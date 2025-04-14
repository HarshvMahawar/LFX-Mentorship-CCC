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

## 🧭 Mapping Summary

### EAR Top-Level Claims

| EAR Field              | Source in Keylime                                             | Description                         |
|------------------------|---------------------------------------------------------------|-------------------------------------|
| `eat_profile`          | Static: `tag:github.com,2023:veraison/ear`                    | Required profile tag                |
| `iat`                  | `verifier_timestamp`                                          | Time of attestation |
| `ear.verifier-id`      | `agent.verifier_id`, `agent.verifier_ip`, `agent.verifier_port` | Info about the verifying entity     |
| `ear.raw-evidence`     | `json_response.results.quote`                                 | Base64 TPM quote                    |

---

### `submods` Mapping

`submods` is a map of appraisals, keyed by the attester (agent ID in Keylime).

#### Format:

```json
"submods": {
  "<agent_id>": {
    "ear.status": "...",
    "ear.trustworthiness-vector": { ... },
    "ear.appraisal-policy-id": "..."
  }
}
```
### Mapping Details

| EAR Claim                    | Keylime Source                                     | Notes                                                                 |
|-----------------------------|----------------------------------------------------|-----------------------------------------------------------------------|
| `submods.<agent_id>`        | `agent.agent_id`                                   | Unique agent identifier                                               |
| `ear.status`                | `json_response.status` or result of verifier logic | `"Success"` → `"affirming"`; failures map to `"warning"` or `"contraindicated"` |

---

### Example `ear.trustworthiness-vector` (AR4SI categories):

```json
{
  "instance-identity": 2,
  "executables": 2,
  "hardware": 2,
  "configuration": 48
}
```
### AR4SI Trust Levels

Values follow AR4SI trust levels:

- `2` = **affirming**
- `48` = **warning**
- `96` = **contraindicated**
- `0` = **none**

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
    "d432fbb3-d2f1-4a97-9ef7-75bd81c00000": {
      "ear.status": "affirming",
      "ear.trustworthiness-vector": {
        "instance-identity": 2,
        "executables": 2,
        "hardware": 2
      },
      "ear.appraisal-policy-id": "urn:keylime:runtime-policy:v1"
    }
  }
}
```
