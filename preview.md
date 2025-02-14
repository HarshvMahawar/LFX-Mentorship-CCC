# Harmonizing Open Source Verifiers with RATS Architecture

## Overview

This project explores the integration of open-source attestation verification frameworks with the **Remote Attestation Procedures (RATS) architecture**. The goal is to harmonize verifiers such as **Veraison** and **Keylime** with the **Entity Attestation Token (EAT)** standard, enabling a unified approach to attestation verification across different platforms.

## What is Remote Attestation?

Remote Attestation (RA) is a mechanism for verifying the trustworthiness of a device or system by generating an **attestation report**. This report contains claims that need to be verified by a trusted verifier. The verification process typically involves:

- **Deserializing and syntax checking** attestation report data models.
- **Verifying cryptographic signatures** using trusted anchors.
- **Checking claims against reference values** obtained from the supply chain.
- **Applying semantic relationships** between claims (e.g., hardware and firmware combinations).

## Components of RATS Architecture

The RATS architecture defines multiple roles involved in remote attestation:

- **Attester:** A device or system that generates an attestation report.
- **Verifier:** The entity that validates the attestation report and produces an attestation result.
- **Relying Party:** A system or entity that makes security-critical decisions based on the attestation result.
- **Reference Values:** Known-good values used to validate attestation claims.
- **Endorsements:** Secure statements vouching for the attester's capabilities.

## Veraison: A Flexible Verifier for RATS

**Veraison** is an open-source project designed to facilitate attestation verification by providing reusable software components. Rather than building a universal verifier for all devices, Veraison offers modular services that can be used to:

1. **Receive and process attestation evidence** from diverse attesters (EAT, TPM, DICE, etc.).
2. **Verify attestation claims** using reference values and cryptographic methods.
3. **Generate attestation results** in various formats (e.g., JSON, CBOR).
4. **Provide attestation results** to relying parties in standardized formats (e.g., EAT, YANG).

### Data Flow in Veraison

1. **Evidence Collection:** Attesters generate evidence based on their platform-specific mechanisms.
2. **Evidence Transmission:** Evidence is sent to the Veraison verifier.
3. **Evidence Verification:** The verifier checks the authenticity of the evidence against policies and reference values.
4. **Attestation Result Generation:** Veraison produces a verification result that states whether the attester is trusted.
5. **Communication with Relying Parties:** The attestation result is sent to relying parties in the preferred format.

## Keylime: TPM-Based Attestation

**Keylime** is an open-source remote attestation framework focusing on TPM-based attestation. It consists of:

- **Agent:** Installed on the attesting device to collect and send TPM-based evidence (PCR values, IMA logs).
- **Verifier:** A server-side component that validates TPM evidence and generates attestation results.

### Integration with Veraison

To harmonize **Keylime with Veraison**, we need:

1. **EAT Support in Keylime:** Keylime should structure TPM evidence as EAT claims.
2. **Veraison as a Keylime Verifier:** Veraison should verify Keylime's TPM-based evidence and generate attestation results.
3. **Python-Based EAR Support:** Since Keylime is Python-based, Veraison's attestation results (EAR) should be available in Python.

## Entity Attestation Token (EAT)

EAT is a standardized format for encoding attestation evidence using:

- **JSON Web Token (JWT)** for human-readable encoding.
- **CBOR Web Token (CWT)** for compact, machine-efficient encoding.

Example EAT Claim Set:

```json
{
    "eat_nonce": "MIDBNH28iioisjPy",
    "ueid": "AgAEizrK3Q",
    "oemid": 76543,
    "swname": "Acme IoT OS",
    "swversion": "3.1.4"
}
```

### Cryptographic Signing

EATs are signed using cryptographic keys (TPM, TEE, Secure Element) with algorithms like:

- **ES256 (ECDSA)**
- **RS256 (RSA)**
- **EdDSA (Ed25519)**

## EAT Attestation Results (EAR)

EAR is used to encode the verifier's result, offering:

- **Trustworthiness Vector:** Standardized trust evaluation.
- **Contextual Information:** Details about the appraisal process.
- **Support for Composite Devices:** Attestation of multiple subsystems.

### Example EAR Structure:

```json
{
    "trustworthiness_vector": {
        "integrity": "passed",
        "freshness": "valid",
        "origin": "trusted"
    },
    "reference_values": "provided",
    "endorsements": "verified"
}
```

## References

- [RATS Architecture](https://datatracker.ietf.org/doc/rfc9334/)
- [Veraison Project](https://github.com/veraison)
- [Keylime Project](https://github.com/THS-on/keylime/tree/rats)
- [EAT Specification (IETF)](https://datatracker.ietf.org/doc/draft-ietf-rats-eat/)
- [EAR](https://datatracker.ietf.org/doc/draft-fv-rats-ear/)


