**Key Principles for Mapping to RATS**

Core principles guiding our RATs mapping:

* **Granularity:** Instead of broad role assignments (e.g., "Verifier Role → Verifier"), we'll pinpoint specific components and their functions within the RATS framework.
* **Data Flow:** We'll analyze how evidence, endorsements, and attestation results are generated, processed, and conveyed, aligning these flows with RATS conveyance models.
* **Standardization Focus:** We'll highlight areas where Keylime, JANE and Veraison can better adopt RATS-recommended standards for data formats, protocols, and interfaces.
* **Actionable Insights:** The mapping should provide clear directions on what aspects of Keylime ,JANE and Veraison could be modified to enhance RATS compliance.

**1. Keylime: Enhanced RATS Mapping**

Here's a breakdown of Keylime's components and a mapping to the RATS architecture, focusing on actionable insights:

* **Attester Role:**
    * **Agent (keylime\_agent):**
        * **RATS Mapping:** Attesting Environment. The agent is the core component that embodies the Attesting Environment. It's responsible for collecting evidence from the platform.
        * **Granular Functions:**
            * TPM quote generation: Aligns with evidence generation using a hardware root of trust.
            * Log collection (UEFI, IMA): Evidence collection from various subsystems.
            * Data Provision: The agent provides collected evidence to the verifier.
        * **Actionable Insights:**
            * **Evidence Representation:** Keylime currently uses various custom and binary formats. Adopting standardized evidence formats like the Conceptual Message Wrapper (CMW) will significantly improve interoperability. This involves structuring the TPM quotes, logs, and other data into a CMW representation.
            * **Evidence conveyance:** While Keylime uses mTLS, defining specific RATS conveyance protocols on top of it will be beneficial.

* **Verifier Role:**
    * **Verifier (keylime\_verifier):**
        * **RATS Mapping:** Evidence Appraisal. The verifier performs the Evidence Appraisal by evaluating the collected evidence.
        * **Granular Functions:**
            * Policy-based verification: This is the core evidence appraisal logic.
            * Runtime attestation: Verifies the runtime state of the system.
            * Attestation result generation: The verifier generates the attestation result based on the policy evaluation.
        * **Actionable Insights:**
            * **Attestation Result Format:** Keylime's current failure-based flags should be replaced with structured attestation results using the EAT format. This involves mapping the outcome of policy evaluations to claims within an EAT structure.
            * **Policy Language:** While Keylime has a policy engine, consider aligning it with emerging standards for attestation policy languages to enhance portability and interoperability.

* **Relying Party Role:**
    * **Verifier (keylime\_verifier):**
        * **RATS Mapping:** Attestation Result Appraisal. In Keylime's architecture, the verifier also takes on the role of making access control decisions based on the attestation results.
        * **Granular Functions:**
            * Enforcement of access control: The verifier enforces access control based on the attestation result.
            * Response mechanisms: The verifier triggers responses based on the verification outcomes.
        * **Actionable Insights:**
            * **Decoupling Policy Enforcement:** Ideally, the policy enforcement logic should be decoupled from the verifier. This would allow external systems or dedicated policy decision points to consume the EAT-formatted attestation results and make access control decisions. This aligns better with the RATS principle of separating verification from policy enforcement.

* **Endorser Role:**
    * **Registrar (keylime\_registrar + keylime\_tenant):**
        * **RATS Mapping:** Endorsement Provisioning. The registrar is responsible for managing trust anchors and metadata, and the tenant is responsible for providing trust anchors, such as the trust anchor for the EK certificate.
        * **Granular Functions:**
            * Trust anchor management: The registrar establishes trust anchors, while the tenant may provide specific trust anchors (e.g., for EK certificates).
            * Metadata management: The registrar manages metadata for attesters.
            * Key binding: The registrar handles AK and EK binding.
        * **Actionable Insights:**
            * **Endorsement Formats:** Ensure that the registrar and tenant can handle and provision endorsements in standardized formats as they emerge. Clarify the division of responsibility for different types of endorsements between the registrar and tenant.

* **Reference Value Provider Role:**
    * **Tenant:**
        * **RATS Mapping:** Reference Value Provisioning. The tenant provides reference values necessary for attestation verification.
        * **Granular Functions:**
            * Reference value supply: The tenant supplies reference values.
            * Policy configuration: The tenant configures attestation policies.
            * Node access management: The tenant manages node access.
        * **Actionable Insights:**
            * **Standardized Policy Representation:** The tenant's configuration of attestation policies should ideally use a standard policy language to improve interoperability.

* **Conveyance Protocols:**
    * **Keylime's REST APIs with mTLS:**
        * **RATS Mapping:** Conveyance. Keylime uses REST APIs with mTLS for secure attestation data exchange.
        * **Actionable Insights:**
            * **Protocol Standardization:** While mTLS provides security, consider defining specific application-level protocols based on RATS reference interaction models. This might involve specifying message formats and exchange patterns for evidence and attestation results.

**2. Jane Attestation Engine: Enhanced RATS Mapping**

the RATS mapping for the Jane Attestation Engine:

* **Attester Role:**
    * **Protocol + Trust Agents (tarzan, ratsd):**
        * **RATS Mapping:** Attesting Environment. The combination of Protocol and Trust Agents in JANE constitute the Attesting Environment.
        * **Granular Functions:**
            * Protocol: Manages attestation sources and handles requests for claims.
            * Trust Agents: Collect attestation measurements from hardware sources (TPM, SGX, etc.) and provide evidence (claims).
        * **Actionable Insights:**
            * **Evidence Normalization:** JANE aims to be technology-agnostic. It is crucial to define a normalization layer within the Trust Agents or Protocol to represent evidence from different sources in a consistent, standardized format (e.g., CMW). This will simplify the Verifier's appraisal process.
            * **Dynamic Attestation:** Enhance the protocol to support dynamic attestation requests, where the verifier can request specific types of evidence or measurements from the attesting environment.

* **Verifier Role:**
    * **Verification:**
        * **RATS Mapping:** Evidence Appraisal. The Verification component performs the Evidence Appraisal by validating claims against predefined rules and reference values.
        * **Granular Functions:**
            * Claim validation: The verification component validates the claims provided by the attesting environment.
            * Rule-based appraisal: The verification component appraises the evidence based on predefined rules.
            * Reference value comparison: The verification component compares the claims against reference values.
        * **Actionable Insights:**
            * **Policy Flexibility:** While JANE uses rules and reference values, consider supporting more expressive and flexible policy languages. This would allow for more complex attestation scenarios and better alignment with policy standards.
            * **Appraisal Context:** Provide a mechanism to include contextual information in the evidence appraisal process. This could include timestamps, location data, or other relevant factors that might influence the verification outcome.

* **Relying Party Role:**
    * **Protocol + Policy Mechanisms + External Interfaces:**
        * **RATS Mapping:** Attestation Result Appraisal. The Relying Party role is distributed across these components in JANE.
        * **Granular Functions:**
            * Protocol: Handles transmission of attestation results.
            * Policy Mechanisms: External policy mechanisms consume attestation results and apply external policies.
            * External Interfaces: Manage connectivity with third-party systems.
        * **Actionable Insights:**
            * **Standardized Result Delivery:** Ensure that the Protocol component can deliver attestation results in standardized formats like EAT. This will facilitate integration with external policy mechanisms and relying parties.
            * **Policy Enforcement Points:** Clearly define the interfaces and protocols for external policy enforcement points to consume and act upon the attestation results. This will promote a clean separation of concerns.

* **Endorser and Reference Value Provider Roles:**
    * **External Providers:**
        * **RATS Mapping:** Endorsement Provisioning and Reference Value Provisioning. JANE relies on external providers for endorsements and reference values.
        * **Actionable Insights:**
            * **Provider Interfaces:** Define clear interfaces and protocols for interacting with external providers of endorsements and reference values. These interfaces should support standard formats for endorsements and reference data.
            * **Trust Anchor Management:** Implement robust mechanisms for managing and validating trust anchors obtained from external providers.

* **Conveyance Protocols:**
    * **Protocol (L6/L7) + L5 (Sessions):**
        * **RATS Mapping:** Conveyance. JANE uses a combination of protocol layers and session management for communication.
        * **Actionable Insights:**
            * **Protocol Clarity:** Clearly define the application-level protocols used for attestation data exchange. Specify message formats, exchange patterns, and security considerations.
            * **Session Context:** Leverage the session management capabilities to maintain context across multiple attestation requests and responses. This can be useful for complex attestation workflows.

**3. Veraison: Enhanced RATS Mapping**

Veraison is already well-aligned with the RATS architecture, but we can still provide a more granular mapping and identify areas for potential enhancement:

* **Attester Role:**
    * **Emulators (veraison/evcli):**
        * **RATS Mapping:** Attesting Environment. Veraison itself doesn't include a full attester implementation but provides emulators for testing.
        * **Granular Functions:**
            * Evidence generation: Emulators are used to generate attestation evidence for different platforms and TEEs.
        * **Actionable Insights:**
            * While emulators are useful for testing, focus on ensuring seamless integration with diverse hardware and software attesters in real-world scenarios. This involves supporting various evidence formats and conveyance methods.

* **Verifier Role:**
    * **Veraison Trusted Services (VTS):**
        * **RATS Mapping:** Evidence Appraisal. VTS is the core component that performs evidence appraisal.
        * **Granular Functions:**
            * Attestation token verification: VTS verifies attestation tokens against endorsements and policies.
            * Policy enforcement: VTS enforces appraisal policies to determine the validity of attestation evidence.
            * Attestation result generation: VTS produces attestation results, often in the form of EAT Attestation Result (EAR) tokens.
        * **Actionable Insights:**
            * **Policy Language Standardization:** Continue to enhance support for standardized attestation policy languages to improve interoperability and flexibility in defining verification rules.
            * **Extensibility:** Maintain the extensibility of VTS to support new attestation schemes and evidence formats as they emerge.

* **Relying Party Role:**
    * **Verification API / EAR CLI:**
        * **RATS Mapping:** Attestation Result Appraisal. Veraison provides mechanisms for relying parties to consume and verify attestation results.
        * **Granular Functions:**
            * Attestation result delivery: The Verification API provides EAR tokens to relying parties.
            * Attestation result verification: The EAR CLI allows relying parties to verify the validity of EAR tokens.
        * **Actionable Insights:**
            * **EAT Ecosystem Support:** Promote wider adoption and support for EAT and EAR tokens in different programming languages and platforms to facilitate seamless integration with relying parties.
            * **Fine-grained Result Access:** Consider providing mechanisms for relying parties to request specific claims or subsets of information from the attestation results, rather than always requiring the full EAR token.

* **Endorser and Reference Value Provider Roles:**
    * **Endorsement Provisioning API:**
        * **RATS Mapping:** Endorsement Provisioning and Reference Value Provisioning. Veraison uses the Endorsement Provisioning API to manage endorsements and reference values.
        * **Granular Functions:**
            * Endorsement management: The API manages endorsements from supply chain actors.
            * Reference value management: The API implicitly manages reference values through the provisioning of endorsements and policies.
        * **Actionable Insights:**
            * **Endorsement and Reference Data Standardization:** Ensure that the Endorsement Provisioning API supports standardized formats for endorsements and reference data to improve interoperability with external providers and consumers of this information.
            * **Explicit Reference Value Management:** Consider adding more explicit mechanisms for managing and retrieving reference values, separate from endorsements, to provide greater flexibility and clarity.

* **Conveyance Protocols:**
    * **Challenge-Response API:**
        * **RATS Mapping:** Conveyance. Veraison primarily uses a challenge-response API as its interaction model.
        * **Actionable Insights:**
            * **Protocol Flexibility:** While challenge-response is a common pattern, ensure that Veraison can support other conveyance protocols and interaction models as needed to accommodate different use cases and environments.
