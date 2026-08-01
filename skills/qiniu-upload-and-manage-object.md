---
name: Upload and manage a Kodo object
description: Upload a file to Qiniu Kodo object storage with an upload token, then stat, copy, move, or delete it via the management API.
api: https://github.com/qiniu/api-specs (storage/)
operations:
  - upload (POST up.qiniu.com, UploadToken)
  - stat (GET rs.qiniu.com /stat/<EncodedEntryURI>)
  - copy_object
  - move_object
  - delete_object
  - modify_object_status
---

# Upload and manage a Kodo object

Use this skill to put a file into Qiniu Kodo and then administer it. Auth,
hosts, pagination and error semantics are captured in
`conventions/qiniu-conventions.yml` and `errors/qiniu-error-codes.yml`.

## Prerequisites
- An AccessKey/SecretKey pair from https://portal.qiniu.com.
- A target bucket and its region host.

## Steps
1. **Mint an upload token.** Build a `PutPolicy` (`scope: "<bucket>"` or
   `"<bucket>:<key>"`, `deadline`, optional `returnBody`/`callbackUrl`) and sign it
   with AK/SK to produce an `UploadToken`. Prefer an official SDK
   (`packages/qiniu-packages.yml`) so signing is handled for you.
2. **Upload.** POST the file to the region upload host (`up.qiniu.com` /
   `up-<region>.qiniu.com`) with the `token` and `key`. On success you get the
   stored `key` and `hash`. Handle `406` (CRC mismatch — retry), `579`
   (upload ok, callback failed), and `701` (resumable context expired — restart
   the block).
3. **Confirm.** Call `stat` on `rs.qiniu.com` with the URL-safe-base64
   `EncodedEntryURI` (`base64url("<bucket>:<key>")`) to read `fsize`, `hash`,
   `mimeType`, `putTime`.
4. **Manage.** Use `copy_object`, `move_object`, `delete_object`, or
   `modify_object_status` (enable/disable) against the management host. These are
   idempotent by fully-qualified key — safe to retry on 5xx.

## Error handling
- `401` invalid signature/token, `403` insufficient permission, `612` key does
  not exist, `614` copy/move target already exists. Log `X-Reqid` from every
  response for support.
