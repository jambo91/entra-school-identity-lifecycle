# Joiner Process

## Objective

The joiner process demonstrates how Northstar Academy creates fictional student and teacher identities and automatically provides the correct group membership and Microsoft Entra ID P2 licence.

## Administrative Roles

- User Administrator — creates and manages users
- Groups Administrator — creates and manages dynamic groups
- License Administrator — assigns product licences to groups

Task-specific roles support the principle of least privilege.

## Test Identities

| Name | Identity type | Job title | Department | Employee ID |
|---|---|---|---|---|
| Olivia Carter | Student | Student | Year 10 | STUDENT-1001 |
| Daniel Reed | Teacher | Mathematics Teacher | Mathematics | TEACHER-1001 |

Both identities are fictional and were created only for this home-lab project.

## Dynamic Student Group

The following security group was created:

```text
DG-All-Students
```

Membership rule:

```text
(user.jobTitle -eq "Student")
```

Olivia Carter was added automatically because her job title matched the rule.

## Dynamic Teacher Group

The following security group was created:

```text
DG-All-Teachers
```

Membership rule:

```text
(user.employeeId -startsWith "TEACHER-") and
(user.accountEnabled -eq true)
```

Daniel Reed was initially added automatically because:

- His Employee ID started with `TEACHER-`.
- His account was enabled.

The `accountEnabled` condition was added to prevent disabled leaver accounts from retaining group-based access and licences.

## Group-Based Licensing

Microsoft Entra ID P2 was assigned to:

- `DG-All-Students`
- `DG-All-Teachers`

The licences were assigned to the groups rather than directly to individual users.

This provides:

- Consistent licence assignment
- Reduced manual administration
- Automatic licensing for qualifying users
- Automatic licence removal when a user no longer qualifies for the licensed group

## Validation

| Test | Expected result | Actual result |
|---|---|---|
| Create Olivia Carter | Student account is created | Passed |
| Create Daniel Reed | Teacher account is created | Passed |
| Evaluate student rule | Olivia joins `DG-All-Students` | Passed |
| Evaluate teacher rule | Daniel joins `DG-All-Teachers` | Passed |
| Assign student licence | Olivia inherits Microsoft Entra ID P2 | Passed |
| Assign teacher licence | Daniel inherits Microsoft Entra ID P2 | Passed |

## Evidence
## Evidence

- [Student dynamic membership](../screenshots/01-dynamic-student-membership.png.jpeg)
- [Student dynamic rule](../screenshots/02-student-dynamic-rule.png.jpeg)
- [Teacher dynamic membership](../screenshots/03-dynamic-teacher-membership.png.jpeg)
- [Teacher dynamic rule](../screenshots/04-teacher-dynamic-rule.png.jpeg)
- [Student group-based licence](../screenshots/05-student-group-based-licence.png.jpeg)
- [Teacher group-based licence](../screenshots/06-teacher-group-based-licence.png.jpeg)


## Security Considerations

- Fictional identities were used.
- Temporary passwords were not stored in GitHub.
- Sensitive tenant information was removed from screenshots.
- Licences were assigned through groups instead of manually.
- The teacher rule excludes disabled accounts.
- Administrative access followed least-privilege principles where possible.
