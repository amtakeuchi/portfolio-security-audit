# Application Security Audit — Portfolio Site

A three-layer security assessment of my personal portfolio site (securi-tee.com) using manual code review, SAST, and DAST. Five vulnerabilities identified and remediated. Zero findings on application code post-remediation.

## Why

I built a portfolio site to showcase cybersecurity work. It would be pretty ironic to have security vulnerabilities on a cybersecurity portfolio. So I audited it.

## The Three Layers

### Layer 1: Manual Code Review

**Tool:** Claude Code

Claude Code read the entire codebase and reported on security issues, bugs, and code quality problems before making any changes. Each fix was reviewed and approved individually.

**Command:**
```
claude
> Review this entire codebase. I want to know:
> 1) Any bugs or errors that could cause pages to not render correctly
> 2) Performance improvements I should make
> 3) Any security issues
> 4) Code quality problems or messy patterns
> Don't change anything yet, just give me the full report.
```

**Findings (5 vulnerabilities identified):**

| Finding | Severity | Status |
|---|---|---|
| Exposed CMS token via NEXT_PUBLIC_ prefix | High | Remediated |
| SSRF via image proxy — wildcard hostname allowing any URL | High | Remediated |
| CORS wildcard on admin routes — any origin allowed | Medium | Remediated |
| Open proxy endpoint — unauthenticated path forwarding | Medium | Remediated |
| Missing security headers — zero headers set at application level | High | Remediated |

**Remediations applied:**

- Moved CMS token to server-side environment variable, rotated token
- Restricted image proxy to only domains actually used for images
- Restricted CORS to specific required origin only
- Added path allowlist to API proxy — only expected OAuth endpoints forwarded
- Implemented security headers on all routes:
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains`
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: SAMEORIGIN`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy: camera=(), microphone=(), geolocation=()`
  - `Content-Security-Policy` with separate policies for main site and admin panel

---

### Layer 2: SAST (Static Application Security Testing)

**Tool:** Semgrep (open source)

Static analysis of the source code at rest, scanning for known vulnerability patterns, insecure code practices, and security anti-patterns.

**Commands:**
```bash
# Install
pip install semgrep

# Set UTF-8 encoding (Windows)
$env:PYTHONUTF8=1

# Run scan excluding third-party CMS-generated files
semgrep --config auto --exclude="public/admin" .
```

**Results:**
```
Scanning 68 files (only git-tracked) with:
✔ Semgrep OSS
  ✔ Basic security coverage for first-party code vulnerabilities.

Ran 246 rules on 68 files: 0 findings.
```

Zero findings on application code. The only alerts in the initial scan (before excluding the admin directory) were in third-party CMS-generated files (minified CodeMirror editor components) — code I didn't write and can't modify.

---

### Layer 3: DAST (Dynamic Application Security Testing)

**Tool:** OWASP ZAP (open source)

Dynamic analysis of the running application, sending thousands of requests testing for SQL injection, XSS, path traversal, CSRF, and dozens of other vulnerability types.

**Setup:**
```
1. Install Java JDK 21 (required by ZAP) — adoptium.net
2. Install OWASP ZAP — zaproxy.org
3. Start local dev server: npm run dev
4. In ZAP: Automated Scan → URL: http://localhost:3000 → Attack
```

**Results:**

| Alert | Risk | Real Finding? |
|---|---|---|
| CSP: Failure to Define Directive with No Fallback | Informational | No — CSP was being implemented at time of scan |
| Authentication Request Identified (2) | Informational | No — ZAP identifying forms, not a vulnerability |
| Information Disclosure - Sensitive Information in URL | Informational | No — ZAP flagging its own internal admin URLs |
| User Agent Fuzzer | Informational | No — testing response to unusual User-Agent strings |
| User Controllable HTML Element Attribute (1,513) | Informational | No — false positives, React auto-escapes output |

Zero real vulnerabilities found. All alerts were informational or false positives.

---

## Summary

| Layer | Tool | Files/Rules | Real Findings |
|---|---|---|---|
| Manual Code Review | Claude Code | Full codebase | 5 (all remediated) |
| SAST | Semgrep | 246 rules, 68 files | 0 |
| DAST | OWASP ZAP | Full site crawl, ~30 min | 0 |

## Tech Stack Audited

- Next.js (App Router)
- TinaCMS (headless CMS)
- Vercel (hosting)
- Cloudflare (CDN/security)

## Tools Used

- **[Claude Code](https://claude.ai)** — AI-powered manual code review
- **[Semgrep](https://semgrep.dev)** — Open source SAST scanner
- **[OWASP ZAP](https://zaproxy.org)** — Open source DAST scanner

## Blog Post

Full write-up with the story behind the audit: [I Security-Audited My Own Portfolio Site](https://www.securi-tee.com/blog/I_Security-Audited_My_Own_Portfolio_Site)
