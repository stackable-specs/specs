---
id: penetration-testing
layer: security
extends: []
---

# Penetration Testing

## Purpose

A penetration test is goal-directed adversarial probing of a running system to discover the vulnerabilities a signature-based vulnerability scanner cannot — chained logic flaws, broken authorization, business-logic abuse, multi-step privilege escalations, lateral movement paths, and the classes of issue an attacker would actually exploit. Without shared rules, "we got pen tested" devolves into theatre: an unscoped engagement that misses the assets that matter, an internal team testing its own code with the same blind spots that wrote it, a one-off test six months before a release whose findings nobody tracks, a tester operating without written authorization (which is itself a federal crime in many jurisdictions), or an automated scanner pointed at production with destructive payloads that take services down. The discipline pays off only when scope and authorization are written down before a packet goes out, an external party is engaged on a documented cadence, the methodology is one of the published frameworks (PTES, OWASP WSTG / MASTG / API Top 10, NIST SP 800-115), every finding carries reproducible evidence and a CVSS-anchored remediation SLA, and a re-test verifies the fix before the issue is closed. This spec pins authorization, scope, cadence, methodology, environment, tooling, evidence, remediation tracking, and disclosure so a pen test produces a load-bearing security signal rather than a PDF that gets filed and forgotten.

## Do

- Establish written Rules of Engagement (RoE) — scope, in-scope assets, out-of-scope assets, time window, contacts — before any test begins.
- Engage an independent third-party tester at least annually for production systems, and before every major release or material architecture change.
- Test against a production-equivalent environment with production-shaped data; restrict production tests to explicitly authorized, time-boxed windows.
- Choose a published methodology (PTES, OWASP WSTG / MASTG / API Top 10, NIST SP 800-115) and cite it in the engagement plan.
- Capture reproducible evidence (request / response, screenshots, payloads, tool versions) for every finding.
- Track every finding in the issue tracker with CVSS, owner, and a CVSS-anchored remediation SLA; require a re-test before close.
- Run automated DAST (e.g. OWASP ZAP, Nuclei, Nettacker) in CI to *augment* human pen tests, not replace them.
- Publish a `/.well-known/security.txt` per RFC 9116 and a coordinated vulnerability disclosure policy.

## Don't

- Run a pen test without written authorization from the asset owner — the legal exposure outweighs any finding.
- Let the same team that built the system own its only security testing.
- Treat an automated vulnerability scan as a substitute for a goal-directed pen test.
- Run destructive payloads (DoS, data deletion, ransom simulation) against production without an explicit, signed authorization that names them.
- Use production customer PII as test data; synthesize or anonymize.
- Close a pen-test finding without re-testing the fix and recording the verification evidence.
- Bury the pen test report in a shared drive — feed every finding into the same issue tracker as first-party defects.

## References

- **spec** `vulnerability-scanning` — sibling security-layer spec for automated, signature-based scanning that pen tests complement
- **spec** `dependency-management` — sibling security-layer spec; dependency CVEs surface here, exploitation chains surface in pen tests
- **spec** `sbom` — sibling security-layer spec; pen test scope cross-references the SBOM for known transitive components
- **external** `https://owasp.org/www-project-web-security-testing-guide/` — OWASP Web Security Testing Guide (WSTG)
- **external** `https://owasp.org/www-project-mobile-application-security/` — OWASP Mobile Application Security (MASVS / MASTG)
- **external** `https://owasp.org/API-Security/` — OWASP API Security Top 10
- **external** `http://www.pentest-standard.org/index.php/Main_Page` — Penetration Testing Execution Standard (PTES)
- **external** `https://csrc.nist.gov/pubs/sp/800/115/final` — NIST SP 800-115: Technical Guide to Information Security Testing
- **external** `https://www.first.org/cvss/v4.0/specification-document` — CVSS v4.0 specification
- **external** `https://datatracker.ietf.org/doc/html/rfc9116` — RFC 9116: A File Format to Aid in Security Vulnerability Disclosure (`security.txt`)
- **external** `https://github.com/OWASP/Nettacker` — OWASP Nettacker (automated reconnaissance and DAST framework)
- **external** `https://www.zaproxy.org/` — OWASP ZAP
- **external** `https://nuclei.projectdiscovery.io/` — Nuclei (template-based vulnerability scanner)
- **external** `https://portswigger.net/burp` — Burp Suite

## Rules

1. Maintain a written Rules of Engagement (RoE) document for every penetration test that names the asset owner, the tester, the scope (in-scope and out-of-scope assets, IPs, domains, accounts), the time window, the agreed test types, the contact for emergencies, and the data-handling rules; do not begin testing without a counter-signed RoE on file.
2. Obtain written authorization from the legal owner of every system, network, and account in scope before a packet is sent; do not rely on verbal agreement, Slack consent, or implied authority.
3. Schedule a third-party penetration test against every production-tier system at least annually; do not let an internet-facing production system go more than a year without an external test on file.
4. Schedule an additional pen test before every major release, before exposing a new public surface, and after every material architecture change (new authentication system, new tenant model, new data store on the trust boundary).
5. Engage an independent party — external firm or an internal red team that does not report into the engineering org being tested — for at least one test per cycle; do not rely solely on the build team's own testing.
6. Cite a published methodology in every engagement plan (PTES for full-scope, OWASP WSTG for web, OWASP MASTG for mobile, OWASP API Security Top 10 for APIs, NIST SP 800-115 for technical guidance); do not run unstructured "I'll poke around" engagements.
7. Test against a production-equivalent environment with production-shaped synthetic data; do not run intrusive tests against raw production except inside a written, time-boxed window approved by the asset owner.
8. Do not seed test environments with real customer PII, real credentials, or copied production data; generate synthetic fixtures or use documented anonymization.
9. Forbid destructive payloads (denial-of-service, data deletion, ransom simulation, persistence implants) by default; allow them only when the RoE explicitly names the payload class, the target asset, the expected blast radius, and a rollback plan.
10. Pin and version-control the tooling used by the engagement (e.g. Burp Suite, OWASP ZAP, Nuclei templates, Nettacker, Nmap, sqlmap, BloodHound) and record the exact tool versions in the report; do not run "whatever happened to be installed."
11. Run automated DAST (Nuclei, OWASP ZAP, Nettacker, or equivalent) on a recurring schedule in CI against staging environments to augment human pen tests; do not treat automated scans as a replacement for human, goal-directed testing. (refs: vulnerability-scanning)
12. Capture reproducible evidence for every finding — full HTTP request and response, exploitation payload, screenshot, video, or terminal transcript, plus the tool version and the timestamp; do not log a finding as "trust me, it works."
13. Score every finding with a current CVSS version (v4.0 or v3.1) and record the vector string alongside the score; do not assign severity by gut feel.
14. File every pen-test finding in the same issue tracker the engineering team uses for first-party defects, with the CVSS score, the affected asset, the recommended remediation, and an owner; do not maintain a parallel spreadsheet that nobody triages.
15. Set a CVSS-anchored remediation SLA (for example: Critical 7 days, High 30 days, Medium 90 days, Low 180 days) and gate releases on overdue findings against the surface they affect; do not let Critical / High findings linger past their SLA without an executive-signed exception.
16. Re-test every remediated finding before closing it, and record the re-test evidence (date, tester, method) on the issue; do not close a finding on the developer's say-so.
17. Treat the pen-test report and raw scan output as confidential — encrypted at rest, access-restricted, retention-bounded — and document chain-of-custody from tester to asset owner; do not email reports as plaintext attachments.
18. Cross-reference every finding against the project's SBOM and dependency graph so transitive-component exploits are tracked alongside first-party issues. (refs: sbom, dependency-management)
19. Conduct a written debrief between tester and engineering team within two weeks of report delivery, walking through every Critical and High finding's reproduction; do not deliver a PDF and consider the engagement closed.
20. Publish a `/.well-known/security.txt` per RFC 9116 with at least `Contact:`, `Expires:`, and `Policy:` fields, and a coordinated vulnerability disclosure policy that names triage SLAs; do not require external researchers to guess where to send a finding.
21. Run a tabletop exercise on incident-response paths exposed by pen-test findings (credential exposure, lateral-movement path, exploitable RCE) at least once per year; do not assume the response plan works because the controls compiled green.
22. Track engagement-level metrics across cycles — count of findings by severity, mean time to remediate by severity, percentage of findings re-tested before close, percentage of overdue SLAs — and review them with security leadership; do not run pen tests without measuring whether the discipline is improving year over year.
23. Brief the on-call team and the asset owner before testing begins (test window, source IPs, expected traffic shape) so a real incident is not mistaken for the test (or vice versa); do not start active testing without a "go" from the on-call rotation.
