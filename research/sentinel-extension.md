# Microsoft Sentinel Extension Lab (Proposed)

## Status
**Design proposal / not part of the original BSides talk unless independently implemented and tested.**

## Objective
Explore whether authorized Meshtastic client observations and supporting host logs can be normalized and investigated in Microsoft Sentinel. This is not direct RF monitoring: telemetry availability depends on the collector and integrations.

## Lab stages
1. Document nodes, roles, firmware versions, client software, and the test topology.
2. Select a supported telemetry source or build a small collector using an authorized interface.
3. Normalize events to fields such as timestamp, pseudonymous device ID, event type, severity, source, and lab run ID.
4. Redact sensitive information. Never send PSKs, private keys, message bodies, or unnecessary precise location.
5. Ingest only through an approved, supported Azure Monitor / Sentinel ingestion path.
6. Validate table and column names in the workspace.
7. Run the example KQL queries in the main README, adapted to the implemented schema.
8. Record what was actually observed, what could not be observed, and false-positive causes.

## Example event types
- `NodeSeen`
- `NodeMissing`
- `ConfigChange`
- `BridgeStatus`

Only emit an event type when a source can actually observe it. Do not infer rogue-node detection or RF attack detection from ordinary client-level telemetry.

## Validation record template
| Field | Value |
|---|---|
| Test ID | |
| Date/time | |
| Firmware / software version | |
| Scenario | |
| Expected result | |
| Observed result | |
| Evidence link | |
| Limitations / false positives | |
| Status (planned / in progress / tested) | |

## Safety and cost
Use only owned or explicitly authorized devices. Azure ingestion and retention can incur costs. Confirm workspace, ingestion, permissions, and retention settings before running the lab.
