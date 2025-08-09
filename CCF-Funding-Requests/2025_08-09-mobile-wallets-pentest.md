# Proposal to Fund the Penetration Test of the iOS and Android Wallet applications

## Proposal
Send **7.2 Billion Qubic** (15,840 USD at $2,200 per billion)) to the address `WKNUEDTSRAQETCUYEACBRQHUYMXBMHHVREVNFEPVGHDQRIKGUFKBTJKGEGJE` to fund a Penetration Test of the Mobile Wallet Apps

> Option 0: No  

> Option 1: Yes, 7.2 Billion Qubic to fund the iOS and Android wallets Penetration Test

---

## Overview

We are conducting the first external security assessment of our iOS and Android wallets. This proactive step follows industry best practices, strengthens Qubic’s reputation as a secure platform, and builds trust to encourage more developers to build on it.

---

## Why Now?

We’ve developed important new features — such as the **in-app mobile browser** and **testnet support** — that are not yet live in production. These features increase the app’s complexity and potential attack surface.

Additionally, community members have raised valid concerns, such as:

- How secure is the wallet app overall?  
- Is there any risk of private keys being stolen?  
- How safe is it to use WalletConnect with the app?

Given these concerns and the upcoming feature rollout, this is the right time for a **proactive security assessment**.

---

## Summary of Proposals

We contacted several reputable security firms and received six proposals, each offering different audit types, timelines, and pricing. Below is a summary of the offers, sorted descending by price.

> **Note**: Provider names are anonymized to respect confidentiality, as we are not authorized to disclose them publicly.

| Option   | Provider   | Type of Assessment           | Duration            | Price (USD) | Key Notes                                                                                          |
| -------- | ---------- | ---------------------------- | ------------------- | ----------- | -------------------------------------------------------------------------------------------------- |
| Option 1 | Provider A | Secure Code Review           | 42 business days    | 84,000      | Marketing add-on; two review layers; senior engineers                                              |
| Option 2 | Provider B | Secure Code Review           | 20 days             | 62,000      | Audited major wallets (e.g., MetaMask); strong mobile app security expertise; do not offer pentest |
| Option 3 | Provider C | Secure Code Review           | 18 business days    | 32,760      | Two senior engineers; blockchain + fintech focus; audited wallets and mobile security projects     |
| Option 4 | Provider A | Penetration Test (gray box)  | 13 business days    | 20,800      | Marketing add-on; two review layers; senior engineers                                              |
| Option 5 | Provider C | Penetration Test (white box) | ~1.5 weeks/platform | 15,840      | One Senior engineer per platform; known for wallet and blockchain audits; mobile app expertise     |
| Option 6 | Provider D | Penetration Test (white box) | 1 week              | 10,000      | Strong blockchain and smart contract audit reputation; less explicit mobile wallet experience      |

**Additional Notes**:
- All proposals cover both **iOS and Android** platforms.
- All include **one round of retesting** after fixes.
- Durations refer to delivery time for the initial report.
- “White-box” and “gray-box” refer to how much the auditor knows before testing. Since the wallet is open source, a white-box penetration test is most appropriate (security code reviews are implicitly white-box).

---

## Proposal Selection

The proposals we reviewed vary in both price and scope — a natural difference between penetration tests and secure code reviews.

Secure Code Review – A detailed, line-by-line analysis of the code, security design, and architecture. This approach is more time-intensive and requires specialized expertise.

Penetration Test – Simulates real-world attacks to assess app behavior, environment, and integration risks. While less exhaustive than a full audit, it identifies key vulnerabilities and high-impact security issues.

After evaluating the available options, we selected Option 5 — a white-box penetration test — as our first step. This method offers the fastest path to uncovering real, exploitable risks, delivers strong value for cost, and provides clear guidance for any future targeted reviews.

We chose this provider because they:

- Have proven experience in blockchain and mobile app security
- Follow a clear, professional delivery process
- Offer a competitive price and timeline (~1.5 weeks per platform)

This approach provides solid assurance of the wallet’s core security while keeping the process efficient in both cost and delivery.


## Future Security Assessment Plans and Considerations

This will be our first formal security review, providing a baseline against which future assessments can be measured. As the codebase evolves to support new SDKs, operating system updates, and feature enhancements, additional reviews will be necessary to maintain a strong security posture.

We anticipate future assessments:

- When major security-impacting features are introduced
- As part of a regular cycle, such as annually during periods of high development activity

Depending on the results of this penetration test, we may proceed with full or feature-specific secure code reviews, or combine multiple assessment methods for broader coverage. If vulnerabilities are found in critical components (e.g., transaction signing logic, WalletConnect integration, mobile platform handling), we will plan targeted assessments for those areas and request additional funding. 

In addition, it is in our roadmap to seek separate funding for a secure code review of the ts-library (Qubic TypeScript library) used not only by our wallets but also by many other projects in the ecosystem. Given its broad adoption, a dedicated review would provide value and assurance across multiple teams and products.

This staged approach ensures resources are allocated efficiently while keeping the wallet’s security at a consistently high standard.