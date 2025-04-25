# Mapping Keylime Attestation Result to IETF EAR

## Scope

We map fields from a Keylime durable attestation result (base64-decoded JSON) into EAR specification.

---

### Keylime Attestation Report Fields

| Keylime Field Name            | Keylime Path                     | Example Value                                 
| :---------------------------- | :------------------------------- | :---------------------------------------------
| `agent_id`                    | `agent.agent_id`                 | `"d432fbb3-d2f1-4a97-9ef7-75bd81c00000"`       
| `verifier_id`                 | `agent.verifier_id`              | `"default"`                                    
| `verifier_ip`                 | `agent.verifier_ip`              | `"127.0.0.1"`                                  
| `verifier_port`               | `agent.verifier_port`            | `"8881"`                                       
| `attestation_status`          | `json_response.status`           | `"Success"`                                    
| `attestation_result_code`     | `json_response.code`             | `200`                                          
| `hash_algorithm`              | `json_response.results.hash_alg` | `"sha256"`                                     
| `encryption_algorithm`        | `json_response.results.enc_alg`  | `"rsa"`                                        
| `signing_algorithm`           | `json_response.results.sign_alg` | `"rsassa"`                                     
| `public_key`                  | `json_response.results.pubkey`   | `"<PEM Public Key Data>"`                      
| `quote`                       | `json_response.results.quote`    | `"<Base64 TPM Quote Data>"`                    
| `ak_tpm`                      | `agent.ak_tpm`                   | `"<Base64 AK Data>"`                           
| `mtls_cert`                   | `agent.mtls_cert`                | `"<PEM Certificate Data>"`                     
| `nonce`                       | `agent.nonce`                    | `"zgI4RH6ER7HHk7vY7bez"`                       
| `tpm_policy`                  | `agent.tpm_policy`               | `"{\"mask\": \"0x400\"}"`                       
| `runtime_policy`              | `runtime_policy`                 | `"<JSON Policy Data>"`                         
| `verifier_timestamp`          | `verifier_timestamp`             | `"04/01/2025, 19:49:28"`                       
| `boottime`                    | `agent.boottime`                   | `""`          
| `operational_state`           | `agent.operational_state`          | `3`           
| `pcr10`                       | `agent.pcr10`                      | `""`          
| `ima_pcrs`                    | `agent.ima_pcrs`                   | `[]`          
| `next_ima_ml_entry`           | `agent.next_ima_ml_entry`          | `0`           
| `learned_ima_keyrings`        | `agent.learned_ima_keyrings`       | `{}`          
| `ima_sign_verification_keys`  | `agent.ima_sign_verification_keys` | `{}`          
| `meta_data`                   | `agent.meta_data`                  | `{}`          

---

## EAR Structure Overview

The EAR token includes:

- `eat_profile`: Identifier for the EAR format  
- `iat`: Issued At timestamp  
- `ear.verifier-id`: Info about the entity performing the appraisal  
- `ear.raw-evidence`: The TPM quote (base64)  
- `submods`: Per-attester appraisal results 

---

## Mapping Summary

### EAR Top-Level Claims

| EAR Field              | Source in Keylime                                             | Description                         |
|------------------------|---------------------------------------------------------------|-------------------------------------|
| `eat_profile`          | Static: `tag:github.com,2023:veraison/ear`                    | Required profile tag                |
| `iat`                  | `verifier_timestamp`                                          | Time of attestation |
| `ear.verifier-id`      | `agent.verifier_id`, `agent.verifier_ip`, `agent.verifier_port` | Info about the verifying entity     |
| `ear.raw-evidence`     | `json_response.results.quote`                                 | Base64 TPM quote                    |

---

### `submods` Mapping

All TPM-related evidence should be placed in a **single submod** named:

```json
"submods": {
  "keylime-tpm": {
    ...
  }
}
```

## Trust Vector Mapping Logic

---

### `instance_identity`

| Condition                                           | TrustClaim                            |
|-----------------------------------------------------|----------------------------------------|
| Quote valid, AK present, no revocation              | `TRUSTWORTHY_INSTANCE_CLAIM (2)`      |
| Quote invalid (e.g., missing or crypto error)       | `UNTRUSTWORTHY_INSTANCE_CLAIM (96)`   |
| Agent identity not recognized (edge case)           | `UNRECOGNIZED_INSTANCE_CLAIM (97)`    |

**→ Sources**: `ak_tpm`, `pubkey`, `json_response.results.quote`, `agent.agent_id`

---

### `hardware`

| Condition                                                     | TrustClaim                            |
|----------------------------------------------------------------|----------------------------------------|
| Quote valid, no hardware revocation detected                   | `GENUINE_HARDWARE_CLAIM (2)`          |
| Known hardware vulnerability (e.g., IMA or PCR mismatch)       | `UNSAFE_HARDWARE_CLAIM (32)`         |
| Quote crypto fails                                             | `CONTRAINDICATED_HARDWARE_CLAIM (96)`|

**→ Sources**: `quote`, verifier decision, `ak_tpm`, TPM endorsement (if available)

---

### `executables`

| Condition                                                                 | TrustClaim                              |
|----------------------------------------------------------------------------|------------------------------------------|
| IMA log verified, no violation                                             | `APPROVED_RUNTIME_CLAIM (2)`            |
| IMA log verified, but warning in runtime policy                           | `UNSAFE_RUNTIME_CLAIM (32)`             |
| IMA missing / policy not applied / agent sent invalid measurements        | `UNRECOGNIZED_RUNTIME_CLAIM (33)`       |
| IMA verification failed due to contradiction                              | `CONTRAINDICATED_RUNTIME_CLAIM (96)`    |

**→ Sources**: `agent.ima_pcrs`, `pcr10`, `runtime_policy`, `next_ima_ml_entry`

---

### `configuration`

| Condition                                        | TrustClaim                             |
|--------------------------------------------------|-----------------------------------------|
| `runtime_policy` exists and passes validation    | `APPROVED_CONFIG_CLAIM (2)`            |
| `runtime_policy` present, passes with warnings   | `NO_CONFIG_VULNS_CLAIM (3)`           |
| Config failure detected (e.g., IMA PCR mismatch) | `UNSAFE_CONFIG_CLAIM (32)`             |
| Runtime policy parsing fails or is null          | `UNSUPPORTABLE_CONFIG_CLAIM (96)`      |

**→ Sources**: `runtime_policy`, `agent.tpm_policy`

---

### `status` (Submod-Level)

| Overall Outcome from Verifier                  | EAR Status        |
|------------------------------------------------|-------------------|
| `status == "Success"`                          | `"affirming"`     |
| `status == "Fail"` or quote/IMA mismatch       | `"warning"`       |
| Quote or crypto check failed                   | `"contraindicated"` |

**→ Source**: `json_response.status`, error handling from verifier (`record.py`)

---


### Example Output (Simplified)

```json
{
  "eat_profile": "tag:github.com,2023:veraison/ear",
  "iat": 1743840568,
  "ear.verifier-id": {
    "developer": "keylime",
    "build": "verifier@127.0.0.1:8881"
  },
  "ear.raw-evidence": "<base64 TPM quote>",
  "eat.nonce": "zgI4RH6ER7HHk7vY7bez",
  "submods": {
    "keylime-tpm": {
      "ear.status": "affirming",
      "ear.trustworthiness-vector": {
        "instance_identity": 2,
        "hardware": 2,
        "executables": 2,
        "configuration": 2,
      },
      "ear.appraisal-policy-id": "urn:keylime:runtime-policy:v1"
    }
  }
}
```
