# COXY: Tokenizing Tertiary Student Credentials On-Chain

<img width="1798" height="862" alt="image" src="https://github.com/user-attachments/assets/cba33ec8-231f-44cc-b58f-e0bcb2ea18f6" />

COXY is a simple experimental app that shows how tertiary student credentials can live securely on the Cardano blockchain. The idea is that every student has an “academic wallet” that holds their learning history as on-chain credentials and progress NFTs, instead of only relying on fragile paper certificates and scattered PDF files.

In this mockup, a student can see their courses, grades and academic milestones in one place. Each completed module or program can be anchored to Cardano as a transaction hash or NFT. This creates a permanent trail of study progress that cannot be lost in a fire, flood, war or forced migration. It is also much harder to fake, which helps reduce academic fraud and fake qualifications.

COXY is designed around Self-Sovereign Identity (SSI). Students can generate an academic DID (Decentralized Identifier) that belongs to them, not to a single university system. The DID links to their on-chain credentials and NFTs. Over time, a student could carry this DID across multiple institutions and countries, building a portable learning profile that they control.

<img width="1788" height="741" alt="image" src="https://github.com/user-attachments/assets/b1a48902-25d6-4c85-af6f-edd76155de0a" />

Privacy and compliance are core concerns. The UI illustrates “privacy modes” and “minimal disclosure” options. Verifiers, such as employers or scholarship committees, should be able to confirm that a student completed a course or earned a degree without seeing all grades and personal details. The mockup hints at GDPR-friendly behavior such as data export requests and consent revocation, even though no real backend is implemented yet. In a real deployment, off-chain storage and on-chain hashes would be combined so that only the minimum information is written to the blockchain.

The app is split into several views. The **Credentials** view shows the student’s DID, wallet address, balance in test ADA, issued credentials and progress NFTs. The **University Dashboard** shows how a university could manage courses and issue new credentials and NFTs to a student’s DID. The **Verifier Portal** simulates how a third party could paste a link, DID or credential ID and check its validity. The **IP Registry** shows how academic intellectual property, such as theses or code, could be hashed and anchored on-chain with ownership linked to a DID.

There is also a **Coxy Wallet** view that acts as a simple academic wallet mockup. It displays test ADA and token balances, a receive address, send and swap forms, and quick actions like buy, sell, delegate, vote and open a DApp center. None of these actions touch a real blockchain. They are here to show how a future integrated wallet could combine everyday token usage with academic credentials in one consistent interface that follows the same themes and privacy settings as the rest of the app.

<img width="1788" height="681" alt="image" src="https://github.com/user-attachments/assets/443d2745-fb7e-41c4-a4d5-75667c90c5f9" />

This repository is a front-end demonstration only. It does not connect to mainnet or testnet, and it does not store real personal data. All balances, hashes, DIDs and verification results are generated in the browser. The goal of COXY is to make the concept of tokenized tertiary credentials easy to see and understand, so that students, universities, builders and funders can imagine how a full SSI-enabled, privacy-aware, Cardano-based credential system might work in the future.

