---
name: Provision a Qiniu IAM sub-account
description: Create an IAM sub-user, issue an AK/SK keypair, and attach a permission policy scoping the account to specific services and actions.
api: https://github.com/qiniu/api-specs (iam/)
operations:
  - create_user
  - create_user_keypairs
  - create_policy
  - modify_user_policies
  - get_user
---

# Provision a Qiniu IAM sub-account

Use this skill to onboard a least-privilege sub-account. All IAM operations live
on `api.qiniu.com` under `/iam/v1/*` and require the root account's AK/SK
`Qiniu` signature (`authentication/qiniu-authentication.yml`).

## Steps
1. **Create the user.** `create_user` — POST `/iam/v1/users` with `alias` and a
   `password`. The response returns the sub-account `iuid`, `root_uid`, and
   `enabled` flag.
2. **Issue a keypair.** `create_user_keypairs` — mint the sub-account's AK/SK so
   it can sign its own API requests. Store the SecretKey securely; it is shown
   once.
3. **Define permissions.** `create_policy` — create a policy binding
   `service_names` (e.g. `kodo`), the allowed `actions`, and the `resources` it
   may touch. Keep it least-privilege.
4. **Attach the policy.** `modify_user_policies` — bind the policy to the user
   (or bind it to a group and add the user to the group).
5. **Verify.** `get_user` and `get_user_policies` to confirm the effective grant.

## Notes
- Disable rather than delete a keypair when rotating (`disable_user_keypair`).
- `403` means the *caller's* policy lacks the IAM action; `401` means the root
  signature failed. Capture `X-Reqid`.
