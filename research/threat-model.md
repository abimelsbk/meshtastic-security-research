# Threat Model: LoRa Mesh Networks with Meshtastic

This document expands the threat-model portion of the BSides Indore 2026 presentation.

## Assets
- Confidentiality of channel and direct messages
- Channel keys and device configuration
- Node identity and trust relationships
- Location and operational metadata
- Availability and reliability of communications
- Bridge hosts and connected systems

## Threat actors
- Passive observer within radio range
- Person with access to a lost or stolen node
- Unauthorized participant with access to shared configuration or keys
- Insider or operator making unsafe configuration changes
- Attacker targeting an optional bridge or associated host

## Trust boundaries
1. Radio boundary between nodes and nearby observers
2. Channel-key boundary between authorized and unauthorized participants
3. Physical boundary around nodes
4. Operator boundary around configuration and key handling
5. Bridge boundary between off-grid radio mesh and Internet-connected infrastructure
6. Telemetry boundary between collector and Microsoft Sentinel workspace

## Risks and mitigations
| Risk | Potential impact | Mitigation |
|---|---|---|
| Public/default key used for sensitive traffic | Unauthorized decryption by parties who know the key | Private keys, controlled distribution, re-keying |
| Node lost or stolen | Exposure of configuration or access | Physical protection, inventory, incident procedure |
| Metadata exposed | Inference of topology, timing, or location | Data minimization, location controls, careful naming |
| Node unavailable | Lost or delayed messages | Baseline testing, redundancy where feasible, recovery plan |
| Unsafe relay configuration | Reduced reliability or unexpected forwarding | Role and hop-limit review, controlled testing |
| MQTT or other bridge misconfigured | Expanded attack surface | Network restrictions, authentication, logging, patching |
| Telemetry collector compromised | False data or sensitive data exposure | Least privilege, validation, redaction, controlled credentials |

## Assumptions and caveats
- The exact attack surface depends on hardware, firmware, configuration, and deployment.
- Encryption does not provide complete anonymity or availability.
- A missing node is not by itself evidence of an attack.
- Findings must be tied to repeatable tests and the specific version/configuration tested.
