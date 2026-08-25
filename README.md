# Microsoft Entra School Identity Lifecycle Management

## Project Status
✅ Completed and validated


## Project Overview

This home-lab project demonstrates an enterprise-style joiner, mover and leaver identity lifecycle for fictional students and teachers using Microsoft Entra ID.

The solution uses identity attributes, dynamic security groups, group-based licensing, account controls and audit logs to provide and remove access throughout a user's lifecycle.

## Business Scenario

Northstar Academy requires a secure and repeatable identity-management process.

The school needs to ensure that:

- New students and teachers receive the correct identities and access.
- Group membership is based on controlled identity attributes.
- Teachers changing departments receive new access automatically.
- Obsolete departmental access is removed automatically.
- Departing users are prevented from signing in.
- Active sessions are revoked during offboarding.
- Product licences are assigned and recovered through groups.
- Administrative activity can be investigated through logs.

## Solution Summary

### Joiner

Fictional student and teacher accounts were created with structured identity attributes.

Dynamic security groups automatically identified:

- Students through the `jobTitle` attribute
- Teachers through the `employeeId` attribute

Microsoft Entra ID P2 licences were assigned through groups rather than directly to users.

### Mover

Daniel Reed was transferred from the Mathematics Department to the Science Department.

Updating his department and job-title attributes automatically:

- Removed Mathematics group membership
- Added Science group membership
- Retained general teacher membership
- Retained the inherited Microsoft Entra ID P2 licence

### Leaver

Daniel's account was disabled and active sessions were revoked.

Testing confirmed that:

- Daniel could sign in before offboarding.
- Daniel could not sign in after offboarding.
- Account disabling was recorded in the audit logs.
- Session revocation was recorded through the refresh-token timestamp.
- Dynamic-rule validation excluded the disabled account.

- After asynchronous processing completed, Daniel was automatically removed from the teacher groups, and his inherited Microsoft Entra ID P2 license was recovered.

## Architecture and Access Model

| Identity condition | Dynamic group | Intended result |
|---|---|---|
| Job title equals `Student` | `DG-All-Students` | Student access and P2 licensing |
| Employee ID begins with `TEACHER-` and account is enabled | `DG-All-Teachers` | General teacher access and P2 licensing |
| Teacher, Mathematics department and enabled | `DG-Mathematics-Teachers` | Mathematics departmental access |
| Teacher, Science department and enabled | `DG-Science-Teachers` | Science departmental access |

## Final Dynamic Membership Rules

### Students

```text
(user.jobTitle -eq "Student")
```

### All Enabled Teachers

```text
(user.employeeId -startsWith "TEACHER-") and
(user.accountEnabled -eq true)
```

### Enabled Mathematics Teachers

```text
(user.employeeId -startsWith "TEACHER-") and
(user.department -eq "Mathematics") and
(user.accountEnabled -eq true)
```

### Enabled Science Teachers

```text
(user.employeeId -startsWith "TEACHER-") and
(user.department -eq "Science") and
(user.accountEnabled -eq true)
```

## Technologies Used

- Microsoft Entra ID
- Microsoft Entra admin centre
- Microsoft 365 admin centre
- Dynamic membership groups
- Group-based licensing
- Microsoft Entra ID P2
- Microsoft Entra audit logs
- Microsoft Entra sign-in logs
- GitHub

## Administrative Roles

- User Administrator
- Groups Administrator
- License Administrator
- Reports Reader

Task-specific roles were identified to support least-privilege administration.

## Test Identities

| Name | Identity type | Initial role | Initial department |
|---|---|---|---|
| Olivia Carter | Student | Student | Year 10 |
| Daniel Reed | Teacher | Mathematics Teacher | Mathematics |

All identities are fictional and exist only in a controlled home-lab tenant.

## Test Results

| Test | Result |
|---|---|
| Student identity created | Passed |
| Teacher identity created | Passed |
| Student dynamic membership | Passed |
| Teacher dynamic membership | Passed |
| Student group-based P2 licensing | Passed |
| Teacher group-based P2 licensing | Passed |
| Mathematics access before transfer | Passed |
| Science access after transfer | Passed |
| Mathematics access removed after transfer | Passed |
| General teacher access retained | Passed |
| Teacher license retained during transfer | Passed |
| Successful sign-in before offboarding | Passed |
| Account disabled | Passed |
| Active sessions revoked | Passed |
| Sign-in blocked after offboarding | Passed |
| Account change recorded in audit logs | Passed |
| Session revocation recorded in audit logs | Passed |
| Disabled user excluded by rule validation | Passed |
| Automatic dynamic-group removal | Passed |
| Inherited license recovery | Passed |

## Project Documentation

- [Joiner process](documentation/01-joiner-process.md)
- [Mover process](documentation/02-mover-process.md)
- [Leaver process](documentation/03-leaver-process.md)
- [Screenshot evidence](screenshots/README.md)

## Troubleshooting Experience

During the project, a department spelling error prevented Daniel from joining the Mathematics group.

The issue was diagnosed using dynamic-rule validation. Correcting the identity attribute allowed membership processing to succeed.

The project also demonstrated that:

- Rule validation and membership processing are separate operations.
- Dynamic membership changes are asynchronous.
- Security-critical offboarding should use immediate controls such as account disabling and session revocation while automated cleanup completes.

## Skills Demonstrated

- Identity lifecycle management
- Joiner, mover and leaver processes
- User attribute management
- Dynamic membership rules
- Group-based licensing
- Least-privilege role selection
- Access validation
- Authentication testing
- Secure account offboarding
- Session revocation
- Audit-log investigation
- Sign-in-log investigation
- Troubleshooting
- Technical documentation
- Security-conscious evidence handling

## Security and Privacy

- Only fictional identities were used.
- No passwords were stored in the repository.
- No access tokens, secrets or private keys were published.
- Full usernames, tenant details and diagnostic identifiers were removed from evidence.
- Disabled accounts were excluded from final teacher-group rules.
- Screenshots were reviewed before publication.

## Home-Lab Disclosure

This project was completed in a personal Microsoft Entra lab environment.

The business scenario, controls, testing process and documentation were designed to reflect real organizational Identity and Access Management practices.
