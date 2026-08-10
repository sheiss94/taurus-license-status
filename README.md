# taurus-license-status

Independent availability-fallback host for **taurus-license**. Served by GitHub
Pages so it does not share a machine, a network, or a failure mode with the
primary licence service on the Taurus VPS.

Each licence has one file here, `<app>__<site>.json`, containing **only** a
status:

```json
{ "status": "active", "ttl": 86400, "generated": "2026-08-10T17:00:00+00:00" }
```

A shipped app that cannot reach the primary service falls back to reading its
file here. If it says `revoked`, the app stops even while the primary is down.
If it says `active`, a previously licensed device keeps running through its grace
window. A device that has never been licensed gets nothing from here — there is
no application to give.

## The one rule

**Never commit an application bundle, a decryption key, or any source to this
repo.** It is public. It carries status and nothing else. The entire point of
the two-host design is that losing the primary must not expose the code, and
this file is what makes that true. If a payload larger than a status object ever
appears here, that is the incident.

## How files get here

Generated on the VPS from the live licence database, then pushed:

```sh
TL_DATA_DIR=/opt/taurus-license/data /opt/taurus-license/bin/license mirror --out <clone>
cd <clone> && git add -A && git commit -m "status: $(date -u +%F)" && git push
```

`license mirror` writes status objects only — it has no access to bundles or
keys by construction. Re-run it after every `activate` / `warn` / `revoke`, or
the fallback keeps answering with the old state for up to a grace window.
