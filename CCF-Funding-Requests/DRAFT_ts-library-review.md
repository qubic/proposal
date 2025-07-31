# Proposal to Fund the Security Code Review of Qubic TypeScript Library

## Proposal

Send **TBC Billion Qubic** to the address `TBC` to fund a Security Code Review of Qubic TypeScript Library

> Option 0: No  

> Option 1: Yes, TBC Billion Qubic

---

## 1. Summary

We seek funding to perform a comprehensive **security code review** of the open-source Qubic Web3 TypeScript libraries, available at:

- [`ts-library`](https://github.com/qubic/ts-library)
- [`ts-library-wrapper`](https://github.com/qubic/ts-library-wrapper)

These libraries are core components of the Qubic ecosystem, enabling developers to interact with Qubic smart contracts and nodes through TypeScript/JavaScript applications. A formal audit is a prudent and proactive step to ensure their continued reliability, security, and support for growing adoption.

> **Note:** The funding requested covers **only the professional services of the external security auditing team**. Qubic developers' time and contributions (e.g., implementing fixes, assisting with testing) are provided voluntarily and are **not included in this budget**.

---

## 2. Motivation

The Qubic ecosystem is growing, and this library plays a vital role in expanding accessibility and developer adoption by providing Web3 capabilities in JavaScript and TypeScript environments.

Security vulnerabilities in this library could lead to:

- Compromised private keys or wallets  
- Malicious transaction signing  
- Data leaks or manipulation  
- Network abuse or denial of service

As these libraries become more widely integrated across applications, a security review will help ensure they remain robust, trustworthy, and aligned with best practices.

---

## 3. Objectives

- Assess the security of the ts cryptographic libraries and wrapper to identify, focusing on:
  - Cryptographic functions and signing logic
  - Key and Secret Management
  - Cross-Platform packaging
  - Dependency Risk Analysis
  - Input Validation
  - Error and Logging controls
  - Entropy and Randomness Sources
  - Wrapper Interface Security (ts-library-wrapper)
- Recommend and optionally implement remediations
- Publish a public audit report for transparency

---

## 4. Scope of Work

- **Repositories:**
  - [`ts-library`](https://github.com/qubic/ts-library)
  - [`ts-library-wrapper`](https://github.com/qubic/ts-library-wrapper)

- **Estimated Size:** ~6900 lines of code (combined)

### Review Excludes:
- Backend services, unless directly coupled with these packages


---

## 5. Deliverables

| Deliverable                                 | Description                                                        | Responsible Party                   |
| ------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------- |
| Security Audit Report                       | Technical report with severity-ranked findings and recommendations | Security Company                    |
| Code Fixes / Remediation Patches (Optional) | Implementation of suggested security fixes                         | Qubic Developers                    |
| Post-Audit Q&A / Verification Support       | Answer questions and confirm remediations post-fix                 | Security Company                    |
| Public Audit Publication (Markdown/PDF)     | Public release of the final report, if approved                    | Qubic Developers + Security Company |

