# Security Policy

The **Enterprise Infrastructure Management (EIM) Research Laboratory**, Telkom University, takes the security of our software, data infrastructure, and user systems seriously. This Security Policy outlines the supported versions, vulnerability reporting procedures, response timelines, and responsible disclosure practices for **Shared CI/CD Framework**.

---

## Supported Versions

We actively provide security patches, dependency updates, and vulnerability fixes for the following releases and branches:

| Version / Branch | Supported | Notes |
| :--- | :---: | :--- |
| `main` | :white_check_mark: | Actively maintained with the latest security fixes |
| `v1.3.0` | :white_check_mark: | Current production release line |
| `< v1` | :x: | End of Life (EOL) — upgrade to latest release |

Consumers pin the floating major tag (`v1`), which always points to the latest supported `v1.x.x` release.

---

## Reporting a Vulnerability

> [!CAUTION]
> **Do not report security vulnerabilities through public GitHub issues, discussions, pull requests, or public chat channels.**  
> Public disclosure exposes users and systems before a patch can be developed and safely distributed.

If you suspect or confirm a security vulnerability in **Shared CI/CD Framework**, please report it using one of the private reporting channels below:

### Option 1: GitHub Private Vulnerability Reporting (Preferred)
1. Go to the repository's [Security Tab](https://github.com/LABEIM/Shared-CICD/security).
2. Click **Report a vulnerability** under **Private vulnerability reporting**.
3. Complete the advisory form with technical details, affected components, and reproduction steps.

### Option 2: Private Security Email
Send an email directly to our security team:
- **Lab Administration:** [lab@eimlab.org](mailto:lab@eimlab.org)
- **Subject:** `[SECURITY] Vulnerability Report in Shared CI/CD Framework: <Brief Summary>`

---

## What to Include in Your Security Report

To help our security maintainers validate and resolve the vulnerability quickly, please include:

1. **Vulnerability Summary:** A clear description of the vulnerability type (e.g., Remote Code Execution, SQL/NoSQL Injection, Authentication Bypass, XSS, CSRF, Insecure Deserialization, Privilege Escalation, or workflow command injection).
2. **Affected Version / Commit:** The specific branch, tag, or commit hash where the vulnerability exists.
3. **Step-by-Step Reproduction:** Exact steps to reproduce the issue from a fresh setup or deployment, including a minimal caller workflow where applicable.
4. **Proof of Concept (PoC):** Sample payload, script, screenshot, or network trace demonstrating the exploit.
5. **Impact Assessment:** Real-world impact if exploited (e.g., secret exfiltration, unauthorized deployment, denial of service).
6. **Remediation Ideas (Optional):** Suggested patch, architectural workaround, or mitigation code.

---

## Response Timeline & Service Level Objectives (SLO)

We make every reasonable effort to meet the following response milestones:

| Stage | Target Timeline | Details |
| :--- | :--- | :--- |
| **Initial Acknowledgment** | **Within 24–48 hours** | We acknowledge receipt of your report and begin initial validation. |
| **Triage & Severity Rating** | **Within 3–5 business days** | We evaluate CVSS score, reproduction viability, and affected systems. |
| **Fix & Patch Development** | **7–21 business days** | A fix is created in a private security advisory branch and tested. |
| **Deployment / Release** | **Immediate upon test pass** | A patched release or hotfix is published. |
| **Public Disclosure** | **Coordinated (30–90 days)** | Public advisory is released in coordination with the reporter. |

---

## Scope & Guidelines

### In-Scope
- Workflow command injection through unsanitized inputs or expressions.
- Secret leakage in logs, artifacts, or rendered output.
- Broken access control in composite actions or reusable workflows.
- Supply-chain risks in third-party actions or dependencies.
- Cryptographic weaknesses or improper key/secret storage guidance.
- Insecure deployment logic (e.g., production/preview branch confusion, URL injection).

### Out-of-Scope
- Denial of Service (DoS / DDoS) attacks against GitHub, Cloudflare, Vercel, or other hosting infrastructure.
- Social engineering, phishing, or physical attacks against lab staff, students, or users.
- Automated scanner reports without a validated, actionable Proof of Concept (PoC).
- Attacks requiring root / administrative access to an already compromised local system.
- Outdated local dependencies where a simple package upgrade already resolves the CVE.

---

## Secure Development & Best Practices

Contributors and maintainers of **Shared CI/CD Framework** follow standard secure engineering practices:

- **Secret Management:** Never commit API keys, private tokens, passwords, or database credentials. Require consumers to provide credentials through GitHub secrets (`secrets: inherit` or organization-level secrets).
- **Input Handling:** Treat all action and workflow inputs as untrusted; pass them through `env:` and quote them in shell steps.
- **Least Privilege:** Keep `permissions:` blocks minimal and avoid `contents: write` unless required for tagging or releases.
- **Dependency Auditing:** Rely on Dependabot, dependency review, and CodeQL to catch known CVEs and unsafe patterns.
- **Signed Commits:** Maintainers sign commits using GPG/SSH to verify code integrity and authorship.

---

## Coordinated Vulnerability Disclosure (CVD)

We strictly adhere to **Coordinated Vulnerability Disclosure (CVD)** principles:
- Please allow reasonable time for our team to resolve the issue before sharing any information publicly.
- In return, we commit to acknowledging your contribution, providing progress updates, and avoiding legal action against researchers acting in good faith.

### Security Hall of Fame & Attribution
We value ethical security researchers and will gladly credit you by name, handle, or organization in our official Release Notes and Security Advisory acknowledgments (unless you prefer anonymity).

---

## Contact & Organization Information

For general inquiries or questions regarding security policies:
- **Lab Contact:** [lab@eimlab.org](mailto:lab@eimlab.org)
- **Organization Website:** [eimlab.org](https://eimlab.org)
- **Institution:** Enterprise Infrastructure Management (EIM) Research Laboratory, Telkom University
