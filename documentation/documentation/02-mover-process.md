# Mover Process

## Objective

The mover process demonstrates how Microsoft Entra ID automatically updates a teacher's departmental access when their user attributes change.

Daniel Reed was transferred from the Mathematics Department to the Science Department.

## Business Requirement

Northstar Academy requires departmental access to follow a teacher's current department.

When a teacher transfers:

- Access to the previous department must be removed.
- Access to the new department must be granted.
- General teacher access must remain available.
- The teacher's existing licence must remain active.
- Administrators should not need to change group membership manually.

## Mathematics Teachers Group

The following dynamic security group represents Mathematics departmental access:

```text
DG-Mathematics-Teachers
```

Membership rule:

```text
(user.employeeId -startsWith "TEACHER-") and
(user.department -eq "Mathematics") and
(user.accountEnabled -eq true)
```

Before the transfer, Daniel matched this rule because:

- His Employee ID started with `TEACHER-`.
- His department was `Mathematics`.
- His account was enabled.

## Science Teachers Group

The following dynamic security group represents Science departmental access:

```text
DG-Science-Teachers
```

Membership rule:

```text
(user.employeeId -startsWith "TEACHER-") and
(user.department -eq "Science") and
(user.accountEnabled -eq true)
```

Before the transfer, Daniel did not match this rule because his department was `Mathematics`.

## Attribute Changes

Daniel's attributes were changed as follows:

| Attribute | Before | After |
|---|---|---|
| Job title | Mathematics Teacher | Science Teacher |
| Department | Mathematics | Science |
| Employee ID | TEACHER-1001 | TEACHER-1001 |
| Account enabled | True | True |

The Employee ID remained unchanged because Daniel was still a teacher.

## Automated Result

After Microsoft Entra processed the attribute change:

- Daniel was removed from `DG-Mathematics-Teachers`.
- Daniel was added to `DG-Science-Teachers`.
- Daniel remained in `DG-All-Teachers`.
- Daniel retained his inherited Microsoft Entra ID P2 licence.
- No manual group-membership changes were required.

## Validation

| Test | Expected result | Actual result |
|---|---|---|
| Mathematics membership before transfer | Daniel is a member | Passed |
| Science membership before transfer | Daniel is not a member | Passed |
| Change department to Science | User attributes save successfully | Passed |
| Mathematics membership after transfer | Daniel is removed | Passed |
| Science membership after transfer | Daniel is added | Passed |
| General teacher membership | Daniel remains a member | Passed |
| Microsoft Entra ID P2 licence | Licence remains active | Passed |

## Troubleshooting

During testing, Daniel was not initially added to the Mathematics group.

### Cause

The `Mathematics` department value contained a spelling error. The user attribute did not exactly match the value required by the dynamic membership rule.

### Resolution

The department spelling was corrected to:

```text
Mathematics
```

After the attribute was corrected and the dynamic rule was processed, Daniel was automatically added to the group.

### Lesson Learned

Dynamic membership rules depend on accurate and consistently managed identity attributes.

Organisations should use:

- Standardised department names
- Controlled identity data sources
- Attribute validation
- Regular group-membership monitoring
- Clear naming conventions

## Evidence

- [Mathematics membership before transfer](../screenshots/07-mathematics-teacher-membership-before-transfer.png.jpeg) 
- [Science membership after transfer](../screenshots/09-science-membership-after-transfer.png)
- [Mathematics membership after transfer](../screenshots/10-mathematics-membership-after-transfer.png)
- [General teacher membership retained](../screenshots/11-all-teachers-membership-after-transfer.png)
- [P2 license retained](../screenshots/12-teacher-licence-retained-after-transfer.png.jpeg)


## Security Considerations

- Departmental access was controlled through dynamic groups.
- General teacher access was separated from departmental access.
- Account status was included in the final group rules.
- Attribute changes automatically removed obsolete access.
- No manual group membership was used.
- Screenshots were reviewed for confidential information.
