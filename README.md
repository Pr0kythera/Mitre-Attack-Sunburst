# MITRE ATT&CK v18 Detection Depth Map

**A native MITRE ATT&CK v18 visualization tool for detection engineers.**

This Proof of Concept (POC) tool moves beyond the traditional "Bingo Card" by visualizing detection depth across the full v18 hierarchy. It allows for multi-layer import, temporal comparison, and granular gap analysis.

## Key Features

* **v18 Native Hierarchy**: Visualizes the complete path: `Tactic → Technique → Sub-technique → Strategy → Analytic → Data Component`.
* **Multi-Layer Management**: Import, toggle, and merge multiple coverage layers (e.g., Visibility + Threat Intel + Active Detections).
* **Layer Comparison (Diffing)**: Compare two timestamped layers to visualize coverage drift, improvements, or regressions over time.
* **Contextual Activation**: Prevents false positive coverage by activating Data Components only within specific detection paths.
* **Gap Analysis**: Identify where you have data visibility (Layer A) and threat intelligence (Layer B) but lack detection rules (Layer C).
* **Local Persistence**: Layers are saved automatically to browser `localStorage`.

## Visualization Hierarchy

Unlike standard navigators, this tool renders the full depth of the v18 data model:

1.  **Tactic** (e.g., Credential Access)
2.  **Technique** (e.g., T1110 - Brute Force)
3.  **Sub-technique** (e.g., T1110.003 - Password Spraying)
4.  **Detection Strategy** (e.g., DET0487 - Detect volume-based failures)
5.  **Analytic** (e.g., AN1336 - Windows Event Logic)
6.  **Data Component** (e.g., DS0028 - Logon Session)

## Usage Guide

1.  **Load**: Open `attack_v18_detection_layers.html` (fetches MITRE STIX data automatically).
2.  **Import**: Drag & drop valid v18 JSON layers (see schema below).
3.  **Analyze**:
    * **Toggle**: Use the sidebar to enable/disable specific layers.
    * **Compare**: Load exactly two layers with timestamps and click **Compare Layers** to view a diff report.
    * **Inspect**: Hover over segments to see layer sources and metadata.

## Detection Layer Schema (v18)

Files must be valid JSON. The tool uses a specialized schema to support v18 primitives.

```json
{
  "name": "SIEM Coverage Q1",
  "version": "1.0",
  "domain": "enterprise-attack",
  "description": "Production detections",
  "timestamp": "2026-01-06T12:00:00Z", // Required for comparison
  "detections": [
    {
      "ruleName": "Brute Force Detection",
      "ruleId": "RULE-001",
      "tactics": ["credential-access"],
      "techniques": ["T1110"],
      "subtechniques": ["T1110.003"],
      "detectionStrategies": ["DET0487"],
      "analytics": ["AN1336"], // Platform-specific logic
      "dataComponents": ["DS0028"], // The actual evidence used
      "score": 1.0,
      "enabled": true
    }
  ]
}
