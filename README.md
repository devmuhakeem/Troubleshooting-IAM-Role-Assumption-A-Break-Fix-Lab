# Troubleshooting IAM Role Assumption: A Break/Fix Lab

A hands-on AWS lab built around a real troubleshooting scenario: a user couldn't assume an IAM role from the Management Console, and I had to diagnose why and fix it without violating least privilege.

## Scenario
Company policy said users must not have permissions attached directly to their identities — instead, they only get permission to assume a role that carries the real permissions. A new user, `Operator-User`, was logging into the console but failing to switch to their assigned role, `Operator-Role`, so they couldn't do their job. My task was to troubleshoot, fix it, and keep the solution strictly least-privilege.

## What I did

### 1. Reproduced the problem first
Signed in as the affected user in a private browser window and tried to switch roles myself. The console returned an "Invalid information in one or more fields" error — confirming the issue before changing anything.

### 2. Diagnosed both sides of the role assumption
Role assumption in AWS needs two things to line up, and either one missing causes the same vague error:
- The **user's identity-based policy** must allow `sts:AssumeRole` on that specific role
- The **role's trust policy** must name that user as a trusted principal allowed to assume it

### 3. Fixed both, scoped tightly
Working as a separate admin user, I corrected the permissions so that:
- The user could assume **only** `Operator-Role` (not any role)
- The role could be assumed **only** by `Operator-User` (not any principal in the account)

I wasn't allowed to create new identity-based policies, so I used a pre-configured customer managed policy instead — a realistic constraint that mirrors how permissions are often managed under governance rules.

### 4. Verified the fix from the user's side
Went back to the affected user's session, retried the role switch, and this time was redirected to the console home page with the console now showing the assumed role as the active identity.

## Key takeaways
- "Can't assume role" is almost always a two-sided problem — checking only the user's permissions or only the role's trust policy misses half the picture
- Least privilege applies to both ends: scoping the user's `AssumeRole` permission to one specific role ARN, and scoping the role's trust policy to one specific user, rather than opening either up broadly just to make the error go away
- Reproducing the failure first and re-testing as the affected user afterward is a much more reliable verification method than just reading the policy JSON and assuming it's right

## Tools
AWS IAM (identity-based policies, role trust policies), AWS Management Console role switching

---
*Completed as an AWS hands-on troubleshooting lab.*
