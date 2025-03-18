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
   - Key Components
   - Mapping to RATS Architecture
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
- **Verification Interface:** Processes and verifies Evidence against reference values.
- **Management Interface:** Manages the configuration and operation of the Verifier.
- **Veraison Trusted Services (VTS):** Core service managing attestation schemes and plugins.
- **KV-Store Interfaces:** Used for data retrieval during verification.
- **Supported Attestation Schemes:** ARM CCA, ARM PSA, TPMs (Parsec, EnactTrust), DICE.

### **Mapping to RATS Architecture**
- **Attester Role:**  
  - Does not directly implement the Attester role. Relies on external agents to provide Evidence in supported formats.
- **Verifier Role:**  
  - Implemented by **VTS**, verifying tokens against endorsements and producing attestation results in the form of EAT Attestation Result (EAR) tokens.
- **Relying Party Role:**  
  - Provides EAR tokens to relying parties via the verification API.
- **Endorser and Reference Values:**  
  - Managed through the **Endorsement Provisioning API**.
- **Conveyance Protocols:**  
  - Uses RESTful APIs aligned with RATS protocols.

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

### **Key Components**
- **Agent:** Runs on the Attester, establishes Attestation Keys (AK), and generates TPM quotes and logs.
- **Registrar:** Manages trust anchors and metadata for agents.
- **Verifier:** Validates evidence against predefined policies and generates attestation results.
- **Tenant:** Configures attestation policies and manages node access.
- **Supported Evidence Types:**
  - TPM Quote with Platform Configuration Registers (PCRs).
  - Integrity Measurement Architecture (IMA) Log.
  - UEFI Measured Boot Eventlog.

### **Mapping to RATS Architecture**
- **Attester Role:**  
  - The **Agent** serves as the Attester, providing evidence such as TPM quotes and IMA logs.
- **Verifier Role:**  
  - Implemented by the **Verifier** component, which appraises evidence based on predefined policies.
- **Relying Party Role:**  
  - Also performed by the **Verifier**, which acts on attestation results.
- **Endorser and Reference Values:**  
  - Managed by the **Registrar** and **Tenant**.
- **Conveyance Protocols:**  
  - Uses secure protocols for component communication.

### **Ongoing Efforts and Gaps**
- Standardizing evidence formats.
- Exploring the use of EAT tokens for attestation results.
- Transitioning to Conceptual Messages Wrapper (CMW) for standardized log formats.

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
