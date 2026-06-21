---
name: deploy-edge-fn-to-box
description: Use when deploying or updating a Supabase edge function to the self-hosted box (db.forj.se / EC2 via the SSM browser shell), especially when the file is larger than ~4KB and a single base64 paste gets mangled.
---

# Deploy an edge function to the self-hosted Supabase box

## Overview

The live Alloy backend (Postgres, Auth, Edge Functions) runs on a **self-hosted Supabase stack on EC2** (`SupabaseAwsStack/Instance`, `i-0f5162624bebb00d8`, eu-north-1, db.forj.se). Only the frontend is on Amplify. The managed Supabase project is a dormant revert target — deploying there does NOT reach production.

You reach the box through **AWS Systems Manager → Session Manager** (a browser shell). Your human partner runs that shell; you cannot drive it (no AWS creds in the agent environment — `aws sts get-caller-identity` returns NoCredentials, and `/plugin`-style installs are unavailable). So your job is to **hand your human partner a paste-ready, self-verifying block**, not to "run the deploy."

Functions live at `/opt/supabase/docker/volumes/functions/<name>/index.ts`. The `supabase-edge-functions` container's `main` router loads them **dynamically per request — no restart needed.** JWT is all-HS256; the browser app's anon key satisfies `verify_jwt`.

## The Iron Gotcha

```
THE SSM BROWSER SHELL CORRUPTS / TRUNCATES A SINGLE PASTED LINE OVER ~4KB.
```

A one-line `printf '%s' '<base64>' | base64 -d | sudo tee` works for small files (~3.5KB pasted clean) but **silently mangles** larger ones (a ~6KB function lost characters twice in one session, and a ~9.5KB SQL blob dropped 2 rows with NO error). Symptoms: `SHA MISMATCH`, a syntax error, or — worst — a clean exit that quietly dropped content.

**Always verify after a box write. Never trust a big single-line paste.**

## Procedure

### 1. Generate a CHUNKED, sha-gated deploy script

From the repo root, base64 the function, fold into short lines, and emit one `printf … >> /tmp/x.b64` per chunk, then decode once. Short lines survive the shell; the sha gate proves byte-exactness against the repo.

```bash
FN=supabase/functions/<name>/index.ts
SHA=$(sha256sum "$FN" | cut -d' ' -f1)
{
  echo 'D=/opt/supabase/docker/volumes/functions/<name>'
  echo 'sudo install -d -m 755 "$D"'
  echo ': > /tmp/fn.b64'
  base64 -w0 "$FN" | fold -w1500 | while IFS= read -r c || [ -n "$c" ]; do
    printf "printf %%s '%s' >> /tmp/fn.b64\n" "$c"
  done
  echo 'base64 -d /tmp/fn.b64 | sudo tee "$D/index.ts" >/dev/null'
  echo "want=$SHA"
  echo 'got=$(sudo sha256sum "$D/index.ts" | cut -d" " -f1)'
  echo '[ "$got" = "$want" ] && echo "SHA MATCH OK <name> ($got)" || echo "SHA MISMATCH got=$got want=$want"'
} > /tmp/deploy.sh
```

**Two bugs that will bite you:**
- The `while read` loop drops the final newline-less chunk unless you add `|| [ -n "$c" ]`. (Verify: extract the chunks back, concatenate, compare to `base64 -w0 "$FN"` — they must be identical.)
- `sudo` is required for the docker socket / the volume path. Plain `docker`/`tee` will fail with permission denied.

### 2. Round-trip verify the script before handing it over

```bash
ORIG=$(base64 -w0 "$FN")
RECON=$(grep -oP "(?<=printf %s ')[^']*" /tmp/deploy.sh | tr -d '\n')
[ "$ORIG" = "$RECON" ] && echo "CHUNKS OK" || echo "CHUNK MISMATCH"
```

### 3. Stage it for your human partner

Load it onto their clipboard so their part is one paste (the deploy script's lines are all short, so it survives):

```powershell
Get-Content /tmp/deploy.sh -Raw | Set-Clipboard
```

Then tell them, exactly: open the SSM session on the box → paste → Enter → expect **`SHA MATCH OK <name>`** → paste the output back.

### 4. Confirm + smoke-test

On `SHA MATCH OK`, the function is live at `https://db.forj.se/functions/v1/<name>` (no restart). Smoke-test from anywhere with the public anon key (`ANON_KEY=` in `/opt/supabase/docker/.env`):

```bash
curl -sS -X POST https://db.forj.se/functions/v1/<name> \
  -H "Authorization: Bearer $ANON" -H "apikey: $ANON" -H 'Content-Type: application/json' \
  -d '{ ... real body ... }'
```

For a SQL apply (not a function), the same chunking + a row-count check after (`select count(*) …`) is mandatory — a clean exit can still have dropped rows.

## Red Flags — STOP

| Thought | Reality |
|---------|---------|
| "I'll just paste the 6KB base64 in one line" | It mangles >~4KB. Chunk it. |
| "I'll run the deploy myself" | No AWS creds in the agent env. Hand your partner a paste block. |
| "SHA wasn't checked but it looked fine" | A clean exit can hide dropped content. Always gate on sha (functions) or row count (SQL). |
| "No need for sudo" | The docker socket + volume path require it. |
| "The managed Supabase project is the backend" | It's the dormant revert target. The box is production. |
| "Edit the managed dashboard / `supabase functions deploy`" | Targets the managed cloud, not the box. Use the SSM paste path. |

## Verification Checklist

- [ ] Chunk lines are each well under 4KB (`fold -w1500`)
- [ ] `|| [ -n "$c" ]` present (last chunk not dropped)
- [ ] Round-trip check passed (chunks rebuild the exact base64)
- [ ] Script ends with a sha256 gate (`SHA MATCH OK`)
- [ ] Smoke-tested the live endpoint after the partner confirms `SHA MATCH`
