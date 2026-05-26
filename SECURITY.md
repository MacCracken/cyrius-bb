# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in cyrius-bb, please report it
responsibly through **GitHub Security Advisories**:

1. Go to the [Security tab](../../security/advisories) of this repository
2. Click **"Report a vulnerability"**
3. Fill in the details and submit

**Do not open a public issue for security vulnerabilities.**

## Scope

This policy covers the cyrius-bb game binary and the files it reads and writes.
cyrius-bb is a single-player game with a deliberately small attack surface:

- **No FFI, no libc, no network.** Everything runs through the Cyrius stdlib
  and the first-party game stack. There is no foreign-function boundary to
  attack.
- **The sandbox is not ours to manage.** Per the AGNOS security model,
  **kavach owns the sandbox, not the application.** cyrius-bb does not roll its
  own security boundary; report sandbox-escape concerns against kavach.
- **Upstream-stack vulnerabilities** (the Cyrius stdlib, mabda, kiran, impetus,
  shravan, sankoch, sigil) belong in their respective repositories. If such a
  bug affects cyrius-bb users specifically, flag it here and we will harden the
  call site, document the workaround, or both.

### Untrusted-input surfaces

The only places external data enters the game:

1. **High-score save file** — `~/.cyrius-bb/scores.cyb` (planned for M5):
   sankoch-decompressed and **sigil-integrity-verified** on load. A hash
   mismatch means a corrupted or hand-edited save — cyrius-bb **refuses to
   load it and does not crash**. Tamper detection is a correctness requirement,
   not a best-effort.
2. **Optional user-supplied music** — `~/.cyrius-bb/music/<level>.ogg`
   (planned for M4): validated before decode; silent-by-default if absent. No
   path traversal — file paths from the data dir are validated, no `../`
   escape.
3. **Command-line arguments** — bounds- and range-checked before use.

Every `var buf[N]` is bounded (N is **bytes**); no `sys_system()` with
unsanitized input.

## Supported Versions

cyrius-bb is **pre-1.0** — the surface is still moving. Only the current
development line receives security fixes; there is no back-port commitment
until the v1.0 release.

| Version | Supported                                              |
|---------|--------------------------------------------------------|
| 0.x     | **Yes** — current development line, receives fixes     |
| < 0.1   | No                                                     |

Once v1.0 ships, this table moves to a standard supported-versions policy.

## Response Timeline

| Action                        | Target                     |
|-------------------------------|----------------------------|
| Acknowledgement               | Within **48 hours**        |
| Initial assessment            | Within **5 business days** |
| Fix for CRITICAL severity     | Within **14 days**         |
| Fix for HIGH severity         | Within **30 days**         |
| Fix for MEDIUM / LOW severity | Next scheduled release     |

Severity ladder: **CRITICAL** (exploitable immediately) / **HIGH** (moderate
effort) / **MEDIUM** (specific conditions) / **LOW** (defense-in-depth) — the
same rubric the internal audits use.

## Audit History

No security audit has run yet — cyrius-bb is at scaffold stage (v0.1.0, no
gameplay code). A **P(-1) hardening pass** runs before the first feature minor
and before the v1.0 freeze, per [`CLAUDE.md`](CLAUDE.md) and the
[first-party standards](https://github.com/MacCracken/agnosticos/blob/main/docs/development/first-party/first-party-standards.md#security-hardening-new--required-before-every-release).
Findings will be filed as `docs/audit/YYYY-MM-DD-audit.md`.

| Date | Release | Findings | Report |
|------|---------|----------|--------|
| —    | —       | none yet | —      |

## Disclosure

We follow coordinated disclosure. Once a fix is released, we will publish a
security advisory crediting the reporter (unless anonymity is requested).
Audit findings that surface internally are disclosed through `docs/audit/*.md`
and the corresponding CHANGELOG entry.
