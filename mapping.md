# **Remote Attestation Procedures (RATS) Architecture Analysis**

This document provides a detailed analysis of the implementation of the Remote Attestation Procedures (RATS) architecture in three open-source projects: **Veraison**, **Keylime**, and **Jane Attestation Engine**. It covers the architecture, key components, role mapping, and ongoing efforts to align with RATS standards for each project.

---

## **Table of Contents**
1. [Introduction](#introduction)
2. [Veraison](#veraison)
   - Overview
   - Architecture Diagram
   - Key Modules
   - Mapping to RATS Architecture
   - Ongoing Efforts and Gaps
3. [Keylime](#keylime)
   - Overview
   - Architecture Diagram
   - Key Components and Mapping to RATS Architecture
   - Ongoing Efforts and Gaps
4. [Jane Attestation Engine](#jane-attestation-engine)
   - Overview
   - Architecture Diagram
   - Key Components and Mapping to RATS Architecture
5. [Conclusion](#conclusion)

---

## **Introduction**
The **Remote Attestation Procedures (RATS)** architecture, defined by the IETF, provides a standardized framework to assess the trustworthiness of devices by evaluating their current state. This document summarizes the RATS architecture, explores its implementation in **Veraison**, **Keylime**, and **Jane Attestation Engine**, and highlights gaps and ongoing efforts to align with RATS standards.

---

## **Veraison**

### **Overview**
**Veraison** is an attestation verification project under the **Confidential Computing Consortium**. It focuses on providing a flexible and extensible Verifier component for remote attestation, supporting multiple attestation token formats and providing APIs for evidence verification and endorsement provisioning.

### **Architecture Diagram**
![Veraison Architecture](https://github.com/HarshvMahawar/LFX-Mentorship-CCC/blob/main/veraison_architecture.png)  
*Figure 1: Architecture of Veraison services.*

### **Key Modules**
- **Provisioning Interface:** Manages endorsements from supply chain actors.
- **Verification Interface:** Processes and verifies Evidence against reference values using trust anchors and appraisal policies.
- **Management Interface:** Manages the appraisal policy for evidence
- **Veraison Trusted Services (VTS):** Core service managing attestation schemes and plugins.
- **KV-Store Interfaces:**
  - Used for data retrieval during verification
  - Storing data during provisioning and management.
- **Supported Attestation Schemes:** ARM CCA, ARM PSA, TPMs (Parsec, EnactTrust), DICE.

### **Mapping to RATS Architecture**
- **Attester Role:**  
  - Implemented in emulators (https://github.com/veraison/evcli)
- **Verifier Role:**  
  - Implemented by **VTS**, verifying tokens against endorsements and producing attestation results in the form of EAT Attestation Result (EAR) tokens.
- **Relying Party Role:**  
  - Provides EAR tokens to relying parties via the verification API.
  - EAR CLI (https://github.com/veraison/ear/tree/main/arc) to verify EARs
- **Endorser and Reference Values:**  
  - Managed through the **Endorsement Provisioning API**.
- **Conveyance Protocols:**  
  - Challenge-response API as interaction model (https://www.ietf.org/archive/id/draft-ietf-rats-reference-interaction-models-13.html#section-7.1).

### **Ongoing Efforts and Gaps**
- Enhancing support for standardized token formats.
- Adding support for implementations of EAR in different programming languages.

---

## **Keylime**

### **Overview**
**Keylime** is a comprehensive remote attestation solution for Linux systems, focusing on TPM-based attestation. It ensures cloud infrastructure trustworthiness by continuously collecting and verifying evidence.

### **Architecture Diagram**
![Keylime Architecture](https://github.com/HarshvMahawar/LFX-Mentorship-CCC/blob/main/keylime_architecture.png)
*Figure 2: Simplified model of the interactions between components of Keylime.*

### **Key Components and Mapping to RATS Architecture**

- **Attester Role → Agent**
   - **Agent:**  
     - Functions as the Attester in RATS, responsible for collecting and transmitting evidence.  
     - Generates TPM quotes and logs.  
     - Provides evidence in various formats:  
       - `TPM2B_ATTEST`, `TPMT_SIGNATURE` (custom TPM format).  
       - Platform Configuration Registers (PCRs) (binary format via `tpm2-tools`).  
       - TCG UEFI Eventlog (binary format from the kernel).  
       - IMA Eventlog (ASCII log format from the kernel).  
       - Public key, boot time, etc.  

- **Verifier Role → Verifier**
   - **Verifier:**  
     - Appraises received evidence against predefined policies.  
     - Generates attestation results, primarily flagging failures.  
     - **Runtime Attestation:**  
       - Evaluates allowed/excluded files, kernel keyrings, verification keys, and IMA-buf entries.  
       - Implements policy-driven verification.  

- **Relying Party Role → Verifier**
   - **Verifier as Relying Party:**  
     - Acts upon attestation results.  
     - Enforces access control and response mechanisms based on verification outcomes.  

- **Endorser Role → Registrar**
   - **Registrar:**  
     - Establishes trust anchors and manages metadata for attesters.  
     - Handles Attestation Key (AK) and Endorsement Key (EK) binding.  

- **Reference Value Provider Role → Tenant**
   - **Tenant:**  
     - Supplies reference values necessary for attestation verification.  
     - Configures attestation policies and manages node access.  
     - Validates EK certificates.  

- **Conveyance Protocols → Secure Messaging**
   - Uses REST APIs for secure attestation data exchange.
   - Implements mutual TLS (mTLS) authentication to ensure encrypted and authenticated communication between Keylime components.

### **Ongoing Efforts and Gaps**
- **Adopting EAT for Attestation Results**  
  - Moving towards **structured attestation results** using the **Trust Vector** model.  
  - Mapping failure types into a **standardized format**.    
- Integrating **Conceptual Messages Wrapper (CMW)** for structured evidence representation.  
- Adopting **TCG Canonical Event Log Format** to unify UEFI, IMA, and user-space logs.
- Registering media types for attestation messages.  .  

---

## **Jane Attestation Engine**

### **Overview**
**Jane Attestation Engine** (a fork and major rewrite of the former A10 Nokia Attestation Engine i.e. NAE) is an experimental remote attestation framework designed to be technology-agnostic.

### **Architecture Diagram**
![JANE Architecture](https://github.com/HarshvMahawar/LFX-Mentorship-CCC/blob/main/jane_architecture_new.png)
*Figure 3: JANE Architecture.*

### **Key Components and Mapping to RATS Architecture**

- **Attester Role → Protocol + Trust Agents + Element Management**
   - **Protocol:**  
     - Acts as the core interface for managing attestation sources.  
     - Handles requests and retrieval of claims, functioning as the "Attesting Environment" in RATS.
   - **Trust Agents (e.g., tarzan, ratsd):**  
     - Responsible for collecting attestation measurements from hardware sources such as TPM, SGX, and other TEEs.  
     - Serves as the primary source of "Evidence" (claims).
   - **Element Management (Maybe):**  
     - Facilitates attestation configuration and process orchestration.

- **Verifier Role → Verification**
   - The "Verification" component in JANE performs "Evidence Appraisal" by validating claims against predefined rules and reference values.

- **Relying Party Role → Protocol + Policy Mechanisms + External Interfaces**
   - **Protocol:**  
     - Handles the transmission of attestation results to external entities.
   - **Policy Mechanisms:**
     - JANE leaves the final trust decision to external policy mechanisms
     - Consumes attestation results and applies external policies for "Attestation Results Appraisal."
   - **External Interfaces:**  
     - Manages connectivity with third-party systems interacting with JANE.

- **Endorser and Reference Value Provider Roles → External Providers**
   - **External Trust Sources:**  
     - JANE integrates with external providers to fetch endorsements and reference values required for verification.

- **Conveyance Protocols → Protocol (L6/L7) + L5 (Sessions)**
   - **Protocol Layer (L6/L7):**  
     - This layer handles the application-level protocols for communication. 
   - **Session Management (L5):**  
     - JANE uses sessions to bind together multiple requests, indicating a layer 5 (session layer) functionality.

---

## **Conclusion**
The RATS architecture provides a robust framework for remote attestation, and the examined projects (**Veraison**, **Keylime**, and **Jane Attestation Engine**) demonstrate significant progress in adopting these standards. Each project aligns well with RATS roles and protocols, but ongoing efforts to standardize formats and improve interoperability are essential for achieving full compliance.

---

## **References**
1. **Veraison:** [GitHub](https://github.com/veraison)  
2. **Keylime:** [Official Website](https://keylime.dev)  
3. **Trustee:** [GitHub](https://github.com/confidential-containers/trustee)  
4. **Jane Attestation Engine:** [GitHub](https://github.com/iolivergithub/jane/)  
5. **Thore Sommer's Master’s Thesis:** [PDF](https://thson.de/static/masters_thesis_thore_sommer.pdf)  
6. **RATS Cheatsheet:** [GitHub](https://github.com/CCC-Attestation/rats-cheatsheet?tab=readme-ov-file#rats-cheat-sheet)  
7. **IETF RATS Endorsements:** [IETF](https://datatracker.ietf.org/doc/draft-ietf-rats-endorsements/)  

---
