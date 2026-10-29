# Beyond the Internet: Practical Security of LoRa Mesh Networks with Meshtastic

**Research and community talk:** Abimel S B Kulumala  
**Original presentation:** BSides Indore 2026 · 6 September 2026  
**Track:** Cyber Physical, IoT & Industrial Security  
**Focus:** LoRa, Meshtastic, off-grid communications, IoT security, threat monitoring  
**Companion material:** [`docs/BSides-Indore-2026-Beyond-the-Internet.pdf`](docs/BSides-Indore-2026-Beyond-the-Internet.pdf)

> **Scope and transparency:** This repository documents the research themes in the original BSides presentation and proposes an optional Microsoft Sentinel monitoring extension. The Sentinel section is a design/lab plan, not a claim that Sentinel integration or a live demo was part of the original BSides talk. Only describe it as implemented after you build and test it.

## Overview

What happens to security when the Internet, cellular networks, and Wi-Fi are unavailable? LoRa-based Meshtastic devices can provide decentralized, low-power messaging and packet relaying, but removing conventional infrastructure does not remove the attack surface.

The central finding of this research is that securing an off-grid mesh is not only a cryptography problem. Configuration, key management, device trust, metadata exposure, physical access, routing behavior, availability, and operational security all matter.

This project is intended for cybersecurity practitioners, IoT and embedded security researchers, penetration testers, and learners exploring wireless and resilient communications.

## Research questions

1. What security properties does Meshtastic provide for channel and direct messages?
2. What can be inferred from metadata even when message payloads are encrypted?
3. How do configuration, key distribution, node roles, and device compromise affect the security of a mesh?
4. How can availability and routing behavior be assessed safely in a controlled lab?
5. How could relevant security events be collected and investigated centrally using Microsoft security tooling, including Microsoft Sentinel, where suitable telemetry and integration paths exist?

## System context

### LoRa, LoRaWAN, and Meshtastic

- **LoRa** is a radio modulation/physical-layer technology for long-range, low-power communication.
- **LoRaWAN** is a networking architecture typically using gateways and network servers for IoT deployments.
- **Meshtastic** is an open-source mesh communication system using LoRa-capable devices to exchange messages and relay packets without requiring conventional Internet infrastructure.

These technologies are related but not interchangeable. The threat model depends on the actual devices, firmware, radio settings, topology, and any bridge to Internet-connected systems.

### Mesh behavior

A mesh allows nodes to relay traffic for other nodes. For example:

`Node A → Node B → Node C → Node D`

Relaying can improve reachability, but every additional node becomes part of the trust and operational picture. Poorly configured, compromised, unavailable, or physically stolen nodes may affect confidentiality, integrity, metadata privacy, or availability.

### Terms used in the presentation

- **Node:** A device participating in the mesh.
- **Channel:** A logical communication group with configuration and an associated encryption key.
- **PSK:** Pre-shared key used for channel encryption.
- **Hop limit:** A limit on packet relaying.
- **NodeDB:** Local database of nodes learned by a device.
- **MQTT bridge:** An optional bridge that may connect mesh traffic to Internet infrastructure and therefore changes the threat model.

## Threat model and attack surface

The original presentation highlights these areas:

| Area | Security questions | Defensive focus |
|---|---|---|
| Channel configuration and keys | Are default or publicly known keys being relied on? Who can access the key? | Generate private keys, control distribution, rotate after exposure |
| Node identity and trust | Can an unknown or unauthorized node participate? | Maintain an inventory and documented onboarding process |
| Device compromise or theft | Can someone access a lost device or its configuration? | Physical protection, secure storage, revocation/rekeying process |
| Metadata and location | What can be inferred from node identifiers, timing, topology, signal information, or location sharing? | Minimize identifying data and disable unnecessary location sharing |
| Routing and availability | Can node loss, congestion, or poor relay settings affect delivery? | Test resilience, choose roles carefully, monitor expected behavior |
| User and operational security | Are keys, screenshots, or configuration files shared insecurely? | User guidance, least privilege, OPSEC procedures |
| Internet bridging | Does MQTT or another bridge expose mesh data to a broader environment? | Document data flows, restrict access, monitor bridge hosts |

This is a defensive threat model, not a claim that every listed condition was observed in every test.

## Encryption and key management

The presentation discusses channel encryption using a pre-shared key and notes that Meshtastic supports AES-128 or AES-256 channel configurations, depending on setup. Nodes communicating on a channel need the relevant key. Direct messaging uses a separate public-key-based protection model described in Meshtastic documentation.

A publicly known or default key must not be treated as a secret. For sensitive deployments:

1. Create a private key using the supported device/application workflow.
2. Distribute it through a controlled channel.
3. Limit who can configure or export keys.
4. Re-key after suspected disclosure or device loss.
5. Document channel membership and ownership.
6. Verify behavior against the version of Meshtastic firmware and documentation in use.

Encryption protects message content under the relevant assumptions; it does not guarantee anonymity, device integrity, correct configuration, or network availability.

## Metadata and privacy

Encrypted communication may still expose useful metadata. Depending on configuration and features, an observer may infer node identifiers, timing, radio activity, routing behavior, network relationships, or location-related information. This can reveal operational patterns without decrypting message contents.

Recommended controls:

- Share location only when required.
- Avoid unnecessary identifying names and public disclosure of topology.
- Review channel and node settings before deployment.
- Treat logs, screenshots, and packet captures as potentially sensitive.
- Set retention and access rules for any collected telemetry.

## Availability and resilience

A decentralized design may reduce dependence on a central access point, but it does not make a mesh immune to outages. Nodes may disappear, become unreachable, be misconfigured, or fail to relay as expected. Excessive or unsuitable relay settings can also create operational problems.

In a controlled lab, record a baseline before testing:

- Known nodes and their intended roles
- Expected routes and delivery behavior
- Firmware and configuration versions
- Message delivery success and latency
- Behavior when a test node is powered off or moved out of range
- Recovery behavior after a node returns

Do not disrupt networks or devices you do not own or have explicit permission to test.

## Defensive deployment checklist

- [ ] Replace publicly known/default channel keys for private deployments.
- [ ] Store and distribute keys securely; define a re-keying process.
- [ ] Maintain a node inventory with owner, role, firmware, and deployment location.
- [ ] Protect devices against theft and unauthorized physical access.
- [ ] Minimize location sharing and identifying metadata.
- [ ] Review router/client/repeater roles and hop limits for the use case.
- [ ] Keep firmware updated and record version changes.
- [ ] Document MQTT or other bridges, including the data that crosses them.
- [ ] Define incident procedures for lost nodes, suspected key exposure, and abnormal behavior.
- [ ] Test resilience in a controlled environment before relying on the mesh operationally.

---

## Microsoft Sentinel extension: from mesh observations to security monitoring

### Purpose

Microsoft Sentinel can provide centralized analysis for **events that can actually be collected and represented as telemetry**. A LoRa/Meshtastic network does not automatically send its radio traffic or node state to Sentinel. A collection and integration component is needed, and the available data will depend on the hardware, software, APIs, logs, and deployment architecture.

This section proposes a follow-on lab that connects the research themes to security operations without implying that the original BSides presentation included a completed Sentinel integration.

### Reference architecture

```text
Meshtastic test nodes
        |
        | authorized observations / supported client or device telemetry
        v
Lab collector (script or application)
        |
        | validate, normalize, redact sensitive fields
        v
Approved ingestion path
        |
        v
Microsoft Sentinel / Log Analytics workspace
        |
        +--> KQL hunting queries
        +--> Analytics rules and incident investigation
        +--> Workbooks / dashboards
```

The collector must use supported interfaces and must not assume that all radio-level events are available. If the collector only sees client-level events, the data and detections must be described as client-level observations rather than full RF visibility.

### Potential event schema

A proposed normalized event could include:

| Field | Purpose |
|---|---|
| `TimeGenerated` | Event timestamp |
| `DeviceId` | Pseudonymous lab node identifier |
| `EventType` | e.g., `NodeSeen`, `NodeMissing`, `ConfigChange`, `BridgeStatus` |
| `Severity` | Lab-defined severity |
| `Source` | Collector or data source name |
| `FirmwareVersion` | Version, if available |
| `ChannelAlias` | Non-sensitive alias; never store channel keys |
| `Details` | Redacted diagnostic context |
| `LabRunId` | Identifier for a controlled experiment |

This is a **proposed schema**, not a built-in Meshtastic or Sentinel table. Implement the ingestion and table mapping before using the sample queries below. Never ingest PSKs, private keys, message contents, or unnecessary location data.

### Example KQL: node missing events

The query below assumes a custom table named `MeshtasticEvents_CL` with the listed columns. Rename fields to match the schema you implement.

```kusto
MeshtasticEvents_CL
| where TimeGenerated > ago(24h)
| where EventType_s == "NodeMissing"
| summarize MissingEvents = count(),
            FirstSeen = min(TimeGenerated),
            LastSeen = max(TimeGenerated)
  by DeviceId_s
| order by MissingEvents desc
```

### Example KQL: configuration change events

```kusto
MeshtasticEvents_CL
| where TimeGenerated > ago(7d)
| where EventType_s == "ConfigChange"
| project TimeGenerated, DeviceId_s, Source_s, Details_s, LabRunId_s
| order by TimeGenerated desc
```

### Example KQL: bridge status changes

```kusto
MeshtasticEvents_CL
| where TimeGenerated > ago(24h)
| where EventType_s == "BridgeStatus"
| project TimeGenerated, DeviceId_s, Severity_s, Details_s
| order by TimeGenerated desc
```

These queries are illustrative and will not work until a custom table or equivalent data model exists and its field names match. They detect only events emitted by the collector; they do not detect radio attacks or rogue nodes unless the telemetry source can identify and report them.

### Suggested lab scenarios

1. **Baseline inventory:** Collect the nodes visible to an authorized lab client and compare with an approved inventory.
2. **Simulated node loss:** Power down a test node and verify whether the collector records a `NodeMissing` event.
3. **Configuration change:** Change a non-sensitive setting on a lab node and verify whether the collector can observe and record the change.
4. **Bridge monitoring:** If an MQTT bridge is intentionally used, monitor its host/service status and relevant access logs.
5. **Investigation:** Use KQL to identify repeated node-loss events or unexpected configuration changes, then review the lab timeline.

These are proposed validation exercises. Mark each scenario as **planned**, **in progress**, or **tested** in your project notes, and publish results only after verifying them.

### Limitations

- Sentinel cannot directly inspect LoRa radio traffic merely because it is connected to a workspace.
- Node disappearance is not proof of malicious activity; range, battery, interference, and normal mobility can produce similar observations.
- A client API may expose only a subset of node or network behavior.
- Custom ingestion, data mapping, retention, and permissions require setup and may incur Azure costs.
- Avoid uploading sensitive message content, keys, personal identifiers, or precise location data unless strictly necessary and authorized.

## Suggested repository structure

```text
meshtastic-security-research/
├── README.md
├── docs/
│   └── BSides-Indore-2026-Beyond-the-Internet.pdf
├── research/
│   ├── threat-model.md
│   └── sentinel-extension.md
├── lab/
│   └── README.md
└── LICENSE
```

The current repository package includes the main README and original PDF. The additional files above are suggested future organization, not files already implemented.

## References and further reading

Use the official project documentation and current product documentation to validate version-specific behavior before testing:

- Meshtastic documentation: https://meshtastic.org/docs/
- Meshtastic source code: https://github.com/meshtastic/firmware
- Microsoft Sentinel documentation: https://learn.microsoft.com/azure/sentinel/
- Azure Monitor Logs / Kusto Query Language: https://learn.microsoft.com/azure/azure-monitor/logs/log-query-overview

## Author

**Abimel S B Kulumala**  
Lead Security Engineer · Cybersecurity researcher and educator

Original presentation: *Beyond the Internet: Practical Security of LoRa Mesh Networks with Meshtastic*, BSides Indore 2026.

## Responsible use

All experiments should be performed on devices and networks you own or are explicitly authorized to assess. This repository is for education, defensive research, and secure deployment. It does not encourage interference with third-party radio networks or unauthorized access to devices.
