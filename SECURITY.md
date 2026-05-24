# Security Policy

We take security issues seriously. Thank you for helping keep Interf safe for the people who use it.

## Supported Versions

Only the latest published release of `@interf/compiler` on npm and the current `main` branch of [`interf-labs/compiler`](https://github.com/interf-labs/compiler) receive security fixes.

## Reporting a Vulnerability

**Please do not open a public GitHub issue for undisclosed security problems.**

Preferred disclosure paths, in order:

1. **Email**: [security@interf.com](mailto:security@interf.com) — primary contact for any security report
2. **GitHub Security Advisories**: use [private vulnerability reporting](https://github.com/interf-labs/compiler/security/advisories/new) on the `interf-labs/compiler` repository
3. **PGP**: encrypted reports accepted on request via the email above

Please include:

- affected version
- impact (what could an attacker do)
- reproduction steps or proof-of-concept
- any suggested mitigation

## Response Commitment

- We will acknowledge your report within **72 hours**
- We will provide an initial assessment within **7 days**
- We will work with you on coordinated disclosure timing — typically a fix released before public disclosure

## Safe Harbor

We support good-faith security research. We will not pursue legal action or law-enforcement investigation against researchers who:

- Make a good-faith effort to avoid privacy violations, data destruction, or service interruption
- Only interact with their own accounts or test environments
- Report vulnerabilities promptly and do not exploit them beyond what is necessary to demonstrate impact
- Give us reasonable time to fix issues before public disclosure

## Scope

Security reports are especially relevant for:

- npm package publish integrity
- unsafe filesystem writes or path traversal
- command execution boundaries (subprocess, agent shell-out)
- accidental leakage of private local files or runtime artifacts
- authentication or authorization bypasses on `interf runtime` / `/v1/*` endpoints
- test or Build Plan files causing unintended code execution
- supply-chain risks in dependencies

## Bug Bounty Program

Interf does not currently run a paid bug bounty program. We will introduce one when scale and product maturity justify it. In the meantime, we publicly acknowledge security researchers who responsibly disclose valid issues (with your permission).

## Trust Center

For our compliance posture, control attestations, and policy documentation see [trust.interf.com](https://trust.interf.com). The Trust Center is the canonical source for SOC 2 status, control implementations, and security questionnaire responses.

## Contact

- Security reports: [security@interf.com](mailto:security@interf.com)
- General inquiries: [info@interf.com](mailto:info@interf.com)
- Trust Center: [trust.interf.com](https://trust.interf.com)
- Trademark questions: see [TRADEMARKS.md](./TRADEMARKS.md)
