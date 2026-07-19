# lex-device-identity

The single home for the **device-certificate + signed-reading envelope** that lets
a tamper-evident trail start **at the device** instead of server-side.

A device generates its own ed25519 keypair and submits only its public key. The
platform issues a **certificate** — `{device_id, tenant, kind, public_key,
issued_at_ms, expires_at_ms}` signed by the deployment key. Each reading the
device sends is signed by its private key. Any service can then verify **offline,
with no registry lookup**:

1. the certificate was issued by the platform,
2. the reading was signed by the certificate's key,
3. the tenant matches and the certificate hasn't expired.

The envelope is ed25519 over a sha256-hex digest of the exact bytes, so signing
and verification agree without any canonicalization step.

## API (`lex-device-identity/src/device_identity`)

| function | side | signature |
|----------|------|-----------|
| `digest(body)` | both | `[crypto] Str` — sha256-hex of the raw body |
| `cert_body(device_id, tenant, kind, public_key, issued_at_ms, expires_at_ms)` | platform | `Str` |
| `issue_cert(device_id, tenant, kind, public_key, issued_at_ms, expires_at_ms, sign_seed)` | platform | `[crypto] Result[Str, Str]` — the signed envelope |
| `verify_reading(cert_env_json, body, reading_sig_b64, platform_pub_b64, now_ms)` | any ingest | `[crypto] Result[VerifiedDevice, Str]` |

`VerifiedDevice = { device_id :: Str, tenant :: Str, kind :: Str }`.

## Who uses it

- **lex-soft** — the platform issues certs (`device_http.lex` → `issue_cert`) and
  can verify readings.
- **lex-telemetry** — verifies device-signed reefer readings at ingest.
- **lex-reefer-edge** — the device that holds a cert and signs each reading.

## Develop

```
lex pkg install
lex check src/device_identity.lex
lex test --allow-effects io,crypto,random tests/
lex fmt --check src/ tests/
```
