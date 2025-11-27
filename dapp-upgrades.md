# COXY: Tokenizing Tertiary Student Credentials On-Chain  
_New desired features for a platform that can serve millions of tertiary students_

## 1. Vision

COXY aims to become a global academic credentials network where every tertiary student owns a secure, verifiable, and portable record of their learning. The goal is to move from a single-site pilot to an infrastructure that can reliably serve millions of students across thousands of universities and colleges.

<img width="1829" height="861" alt="image" src="https://github.com/user-attachments/assets/2cf4d99f-edd4-4e17-9dd3-7e4622bc8d2b" />

In this vision, every grade, project, and intellectual-property output can be anchored on the Cardano blockchain and represented as tokenized assets governed by the COXY token. Students manage their identities and records through the Coxy Academic Wallet, while universities issue credentials through dedicated dashboards, and employers verify them through a simple web portal.

## 2. Current Platform Snapshot

![Coxygen dashboard and wallet](https://github.com/user-attachments/assets/ae675a09-ac5a-4a3b-98fa-ea404ec6f494)

![Reports dashboard](https://github.com/user-attachments/assets/fcdf271c-fc2b-4d8c-8e79-0a325b96cd7b)

The existing Coxygen university dashboard and wallet (as shown in the enrollment and old dApp screenshots) already support a working end-to-end flow for a smaller cohort of users.

- University portals list students, courses, and Haskell Plutus training tasks. Each row represents a unit of learning with an associated identifier and description.  
- The built-in wallet panel can hold test ADA (tADA), send and receive funds, generate new addresses, mint basic tokens, manage mnemonics, and service UTXOs.  
- An enrollment view shows real student records across African universities, including name, qualification, institution, address, and join date, which demonstrates that the system already onboarded hundreds of students.  
- A performance report view aggregates “progress tokens,” displaying total tokens minted, total “developers” trained, and leaderboards of universities and students by on-chain activity.

These components prove that the model works technically and educationally. The next step is to generalize and harden the architecture so that it can support millions of students with production-grade security, UX, and governance.

## 3. High-Level Architecture for Scale

![High-level architecture](https://github.com/user-attachments/assets/a85e2569-6a05-4f35-aa6b-bc67872858a3)

The high-level system diagram introduces a modular architecture designed to scale.

- On the **Student side**, a Student dApp connects to the **Coxy Academic Wallet**. The wallet manages keys, DIDs, and verifiable credentials (VCs) while interacting with backend APIs over HTTP.  
- On the **University side**, an **Admin Dashboard** lets university staff issue and manage credentials.  
- On the **Employer / Verifier side**, a **Verifier Portal** allows employers to check credentials without needing to understand blockchain details.  
- In the **Backend Services** zone, an **API Gateway / BFF** (backend for frontend) coordinates requests to specialized services:
  - An **Identity Service** issues and validates DIDs and verifiable credentials.
  - A **Credential Service** creates and anchors credentials on-chain.
  - An **IP Registry Service** manages intellectual-property registrations linked to theses and projects.
  - A **Notification Service** sends emails and other alerts.
- **Off-chain storage** holds encrypted VC data and optionally IPFS or object storage payloads.  
- On the **Cardano blockchain**, several smart contracts act as anchors and token policies:
  - A **Credential Anchor Contract** stores hashes that prove credential existence and status.
  - A **Progress NFT Policy** mints tokens representing learning progress.
  - An **IP Registry Contract** handles student intellectual property.
  - A **COXY Token Policy** governs the fungible COXY token supply and on-chain fee logic.

This separation of concerns makes it possible to horizontally scale each backend service, add specialized indexes, and integrate with multiple Cardano nodes and storage layers as user numbers grow.

## 4. Credential Issuance Flow

![Credential issuance sequence](https://github.com/user-attachments/assets/83239515-c6f1-4643-9a50-13aa1a723963)

The credential issuance sequence diagram describes how a grade or qualification becomes an on-chain asset.

1. A **University Admin** uses the **Admin Dashboard** to select a student, module, and grade.  
2. The dashboard sends an `IssueCredentialRequest(studentDid, claims)` to the **API Gateway**.  
3. The gateway builds a verifiable credential object and performs validation.  
4. It forwards the unsigned VC to the **Identity Service**, which signs it using the university’s DID key, producing a VC with `vcHash`, `studentDid`, `issuerDid`, and metadata.  
5. The signed VC and associated metadata are sent to the **Credential Service**, which submits a transaction to the **Credential Anchor Contract** on Cardano.  
6. The chain returns a transaction hash and an anchor UTXO reference, which become the immutable anchor for that credential.  
7. The Credential Service mints a **Progress NFT** representing this credential or learning milestone, then records its mint transaction hash.  
8. A “VC envelope” (which could be a deep link or QR code) is generated and passed to the **Coxy Academic Wallet**.  
9. The wallet stores the encrypted VC locally on the student’s device, linking it to their DID and academic profile.  
10. The **Notification Service** sends an issuance email with transaction hashes and a link or QR code to the student for backup and sharing.

For millions of students, this flow must be heavily optimized and batched. Desired enhancements include bulk issuance tools, parallel transaction submission, and automated retry logic with clear status dashboards for university staff.

## 5. Credential Verification Flow

![Credential verification sequence](https://github.com/user-attachments/assets/0ce3a147-024d-4dd5-bfa7-3a51e00eb258)

The verification sequence diagram focuses on the experience of an employer or verifier.

1. An **Employer** begins by pasting a verification link or the student’s DID into the **Verifier Portal**.  
2. The portal sends a `VerifyCredentialRequest(payload)` to the **API Gateway**.  
3. The gateway reads the credential anchor from the **Credential Anchor Contract** via the **Cardano node** and returns anchor data such as hash, issuer, student, and status.  
4. If the verification request includes a full VC presentation, the gateway can immediately validate the VC and its signature.  
5. If no presentation is attached, the system requests one from the student.  
6. The Verifier Portal asks the student, via QR code or link, to share the relevant credential.  
7. The **Coxy Academic Wallet** receives this request and allows the student to build a selective-disclosure VC presentation that reveals only the required claims.  
8. The wallet sends the signed presentation back through the gateway.  
9. The gateway validates the VC, its signature, and its match with the on-chain anchor, then computes the validation result.  
10. Finally, the Verifier Portal presents a clear status and summary to the employer, including whether the credential is valid, revoked, or superseded.

At large scale, this flow must remain extremely simple for employers. Desired features include support for single-click “magic links,” integration with HR systems, and high-availability verification endpoints with caching and rate-limiting.

## 6. Credential Lifecycle and Revocation

![Credential lifecycle state diagram](https://github.com/user-attachments/assets/9b3b606c-6b93-4d7b-8b8c-4e6079fb97ab)

The credential lifecycle state diagram defines how a credential can evolve over time.

- A credential starts as “VC created but not anchored.”  
- Once the anchor transaction is confirmed, its state becomes **Credential valid**.  
- At any time, a **RevokeRequested** event can move it to **Credential invalid**, for example when a degree is rescinded or a credential is issued in error.  
- A **SupersedeRequested** event can mark the credential as **Replaced by new credential**, such as when a corrected transcript is issued.  

This lifecycle is critical when millions of records are in circulation. New features here include time-based revocation proofs, institution-level governance controls, and transparent audit logs so that employers always know why a credential changed state.

## 7. Wallet Onboarding and DID Creation

![Wallet onboarding flow](https://github.com/user-attachments/assets/610cc47f-5ebf-4d17-b6e6-35666f06f120)

The wallet onboarding flow shows how every student gains a self-sovereign identity.

When a student opens the **Coxy Academic Wallet** for the first time, they choose either **New Wallet** or **Restore**.

- For a **New Wallet**, the app generates Cardano keypairs and a mnemonic, walks the student through PIN creation, and generates a DID keypair. It then creates a DID document locally and sends a registration request to the **Identity Service**, which anchors the DID. The wallet stores the DID reference, displays an “Academic Identity” profile, and becomes ready to receive credentials and NFTs.  
- For **Restore**, the student enters an existing mnemonic and PIN. The wallet reconstructs their keys and DIDs, stores the references, and re-shows their academic profile.

To support millions of students, the desired enhancements include mobile-first UX, biometric unlock options, recovery workflows that do not expose the mnemonic, and integration with cloud-based encrypted backups (while keeping private keys under student control).

## 8. Intellectual Property Registration for Theses and Projects

![IP registration flow](https://github.com/user-attachments/assets/16bd7cf9-af31-4b14-bbe6-188585ef1554)

The IP registration flowchart introduces a powerful new feature: tokenizing student research outputs.

The process starts inside the wallet or dApp when a student decides to register a thesis or project as intellectual property.

1. The student selects the work to register.  
2. The system computes a SHA-256 hash of the content, ensuring that the actual document can remain off-chain while its integrity is provable.  
3. The student enters metadata such as title, type, abstract, co-authors, and links.  
4. The system checks whether all co-authors have confirmed the registration. If not, it prompts the remaining co-authors.  
5. Once all co-authors confirm, the system builds an IP token mint transaction with the metadata.  
6. The transaction is submitted to the **IP Registry Contract**. If confirmation fails, the UI shows an error and prompts a retry.  
7. If the transaction succeeds, the wallet stores the IP reference in the profile, notifies the student and co-authors, and exposes the IP in both the academic profile and the IP registry.

This feature allows millions of students to cryptographically prove ownership of their research, creating a global registry of academic innovation. Future enhancements may include licensing options, revenue-sharing rules, and links to journals or research repositories.

## 9. COXY Token and Fee Flow

![COXY token and fee flow](https://github.com/user-attachments/assets/6bde5058-ff61-4838-a3aa-2568358128e7)

The fee and COXY token flow diagram defines how actions are monetized and how value is shared.

Whenever a credential issuance or verification occurs, the system calculates a fee denominated in both ADA and COXY. A transaction is built and submitted to Cardano.

- If the transaction is not confirmed, an error is returned and the frontend guides the user through retries.  
- If the transaction succeeds, an on-chain script executes a **fee split**:
  - One share goes to the **Protocol Treasury**, which funds future development and maintenance. Treasury balances are reflected in analytics dashboards.  
  - Another share goes to the **Institution or Verifier**, rewarding universities and employers for participating in the network.  
  - An optional portion of COXY may be burned or locked, reducing circulating supply and supporting token value.

At scale, this model allows COXY to become the economic backbone of the credential network. Desired features include dynamic pricing based on region, student subsidies funded by the treasury, and transparent, public tokenomics dashboards.

## 10. Analytics and Performance Reporting

![Performance reports](https://github.com/user-attachments/assets/d037198b-3bd6-431d-9031-5086e3b13a71)

The performance reports screenshot demonstrates an early version of the analytics layer.

- It displays the total number of progress tokens minted and the total number of developers trained.  
- It ranks universities and colleges by the number of students and by total on-chain transactions, highlighting particularly active institutions.  
- It shows leaderboards of individual students with at least ten transactions, making engagement visible and motivating.

To support millions of students, analytics must become more comprehensive and near real-time. Desired improvements include institution-level dashboards, geographic heatmaps, retention and completion statistics, and exportable reports for accreditation bodies and funding agencies.

## 11. UX and Product Features for Millions of Students

To move from a successful pilot to a global platform, COXY needs several product-level enhancements beyond the flows already modeled.

1. **Mobile-first, low-bandwidth design**  
   The Coxy Academic Wallet and student dApp should be available as Android and iOS apps, as well as lightweight progressive web apps. All screens should be optimized for intermittent connectivity and small screen sizes so that students in bandwidth-constrained regions can fully participate.

2. **Multi-language support**  
   The platform should offer translation of the wallet, dashboards, and verifier portal into the major languages used by participating countries, starting with English and French and then adding more languages over time.

3. **Institution self-service onboarding**  
   Universities and colleges should be able to onboard themselves, create institutional DIDs, configure issuance templates, and manage roles for admins and registrars without manual intervention from the COXY core team.

4. **Bulk issuance and automation**  
   Registrars should be able to upload CSV files or integrate via APIs to issue thousands of credentials in a single operation. Automated batches can be scheduled for end-of-semester grade releases.

5. **Student self-enrollment**  
   Students should be able to download the wallet, complete identity verification steps defined by their institution, and link their student number to their DID before any credentials are issued.

6. **Privacy-preserving selective disclosure**  
   The wallet should make it easy for students to share only the data an employer needs (for example, degree name and graduation year, but not GPA) using standard selective-disclosure VC presentations.

7. **Security and compliance**  
   All flows must be hardened with rate limiting, audit logs, institutional approval workflows, and compliance with data-protection regulations. For millions of users, automated monitoring and anomaly detection become essential.

![Tools view](https://github.com/user-attachments/assets/361a91b5-ba02-48da-a397-c34655e10335)

Tools

![Universities tools](https://github.com/user-attachments/assets/12a90a38-4e12-4256-9a80-7527891d9b1d)

Enrollment and Colleges

![Enrollment cards](https://github.com/user-attachments/assets/f5eb9e09-1f01-4c29-a828-47849c47e5fb)

## 12. Roadmap Summary

The diagrams and current dashboards show that COXY already has the foundations of a complete on-chain academic credential system: issuance, verification, lifecycle management, wallet-based identity, IP registration, and tokenized economics.

<img width="1795" height="745" alt="image" src="https://github.com/user-attachments/assets/c0cd7407-1538-447c-8ef8-dffed35db5b1" />

The next phase focuses on turning these capabilities into a robust platform for millions of tertiary students by:

- Scaling the underlying architecture and Cardano integration.  
- Simplifying the user experience for students, institutions, and employers.  
- Expanding features such as IP registration, advanced analytics, and COXY token economics.  
- Providing strong governance and revocation models to maintain trust over time.

If executed well, **“COXY: Tokenizing Tertiary Student Credentials On-Chain”** can evolve from a specialized Haskell Plutus training environment into a global infrastructure for academic recognition and innovation.
