# MITRE ATT&CK v18 Detection Depth Map - Multi-Layer Edition

## Preamble

This tool is very much POC and not security tested or ready for enterprise use. My intention is to add functionality and improve it over time to improve usability and robustness. Currently heavily relies on LLM generated codebase as I wanted to quickly iterate on ideas to get a functioning POC ready.

## Overview

This tool provides native MITRE ATT&CK v18 visualization with **multi-layer detection import** capability, designed specifically for detection engineers working with modern SIEM CI/CD pipelines.

Unlike traditional ATT&CK Navigator tools that focus on Technique-level coverage, this visualization embraces the **full v18 hierarchy**:

```
Tactic → Technique → Detection Strategy → Analytic → Data Component
```

This granular approach solves the "Bingo Card Fallacy" by showing not just *what* techniques are covered, but *how deeply* they're covered across different platforms, detection strategies, and data sources.

## Key Features

### 1. **v18 Native Architecture**
- Full support for Detection Strategies (DETxxxx)
- Full support for Analytics (ANxxxx)
- Full support for Data Components (DSxxxx)
- Platform-aware coverage (Windows, Linux, Cloud, SaaS)

### 2. **Multi-Layer Detection Management**
- Import multiple detection layers simultaneously
- Toggle layers on/off to compare coverage
- Union-based merging (any layer activates = active)
- Layer-specific color coding for visual differentiation

### 3. **SIEM CI/CD Integration**
Export detection metadata from your SIEM pipeline in v18 format and import directly into the visualization to see real-time coverage.

### 4. **Gap Analysis Workflow**
Import three complementary layers to identify detection priorities:

**Layer 1: Log Visibility** - What you *can* detect based on ingested log sources  
**Layer 2: Threat Intelligence** - What you *should* detect based on threat actor TTPs  
**Layer 3: Active Detections** - What you *do* detect (current SIEM rules)

**Gap = Visibility ∩ Threat Intel - Active Detections**

Where you have the data AND the threat is real, but no detection exists.

### 5. **Persistent Storage**
- Layers saved to browser localStorage
- Export layers back to JSON for sharing
- Multi-layer export for full environment snapshots

## Detection Layer Schema (v18 Format)

```json
{
  "name": "SIEM Detection Coverage",
  "version": "1.0",
  "domain": "enterprise-attack",
  "description": "Production detection rules",
  "detections": [
    {
      "ruleName": "High Volume Authentication Failures",
      "ruleId": "SENT-BF-001",
      "tactics": ["credential-access"],
      "techniques": ["T1110"],
      "subtechniques": ["T1110.003"],
      "detectionStrategies": ["DET0487"],
      "analytics": ["AN1336", "AN1338"],
      "dataComponents": ["DS0028"],
      "tables": ["SigninLogs", "SecurityEvent"],
      "score": 1.0,
      "enabled": true
    }
  ]
}
```

### Schema Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ruleName` | String | Yes | Human-readable name of the detection rule |
| `ruleId` | String | Yes | Unique identifier from your SIEM (e.g., Sentinel rule ID) |
| `tactics` | Array | Yes | MITRE tactic short names (e.g., "credential-access") |
| `techniques` | Array | Yes | Technique IDs (e.g., ["T1110"]) |
| `subtechniques` | Array | No | Sub-technique IDs (e.g., ["T1110.003"]) |
| `detectionStrategies` | Array | Yes | v18 Detection Strategy IDs (e.g., ["DET0487"]) |
| `analytics` | Array | Yes | v18 Analytic IDs (e.g., ["AN1336"]) |
| `dataComponents` | Array | Yes | v18 Data Component IDs (e.g., ["DS0028"]) |
| `tables` | Array | No | SIEM-specific table/log names for reference |
| `score` | Float | No | Detection quality score (0.0-1.0), default 1.0 |
| `enabled` | Boolean | No | Whether this detection is active, default true |

**Important:** All ID arrays support multiple values because a single detection rule often covers multiple techniques or uses multiple analytics.

### Contextual Activation (Critical Feature)

The tool implements **contextual activation** to prevent false coverage reporting. When you specify a Data Component, it's only activated **within the specific hierarchy path** you define:

```json
{
  "techniques": ["T1059"],           // Only under this technique
  "detectionStrategies": ["DET0516"], // Only under this strategy
  "analytics": ["AN1428"],            // Only under this analytic
  "dataComponents": ["DC0032"]        // Activate DC0032 ONLY in this context
}
```

**Why this matters:**

Data Components are **shared across many techniques**. For example, `DC0009` (Process Creation) is used for:
- T1003 (Credential Dumping)
- T1059 (Command Execution)  
- T1055 (Process Injection)
- ...and many more

Without contextual activation, marking DC0009 as "covered" would make it appear green across **all** techniques that use it, even though your rule only detects credential dumping.

**Contextual activation ensures:**
- DC0009 only shows as covered under T1003 → DET0488 → AN1339
- Other techniques using DC0009 remain red (uncovered)
- Accurate representation of actual detection depth

**Hierarchical matching rules:**

1. **All IDs specified**: Activates only the exact path
   ```json
   // Activates DC0032 ONLY under T1059 → DET0516 → AN1428
   {
     "techniques": ["T1059"],
     "detectionStrategies": ["DET0516"],
     "analytics": ["AN1428"],
     "dataComponents": ["DC0032"]
   }
   ```

2. **Partial hierarchy**: Matches any path containing the specified components
   ```json
   // Activates DC0032 under T1059 but across ANY strategy/analytic
   {
     "techniques": ["T1059"],
     "detectionStrategies": [],
     "analytics": [],
     "dataComponents": ["DC0032"]
   }
   ```

3. **No hierarchy constraints**: Activates globally (not recommended)
   ```json
   // Activates DC0032 everywhere - causes false coverage!
   {
     "techniques": [],
     "detectionStrategies": [],
     "analytics": [],
     "dataComponents": ["DC0032"]
   }
   ```

**Best practice:** Always specify at least `techniques` + `analytics` to ensure contextual accuracy.

## Usage Guide

### Quick Start

1. Open `attack_v18_detection_layers.html` in a modern browser
2. Wait for MITRE v18 STIX data to load
3. Drop a detection layer JSON file or click the upload zone
4. Toggle layers on/off to explore coverage

### Example Workflows

#### Workflow 1: SIEM Coverage Assessment

```bash
# Export your SIEM rules to v18 format
python export_sentinel_rules_to_v18.py > my_siem_coverage.json

# Import into visualization
# Drag and drop my_siem_coverage.json into the tool
```

#### Workflow 2: Multi-Layer Gap Analysis

```
Step 1: Import "Log Visibility" layer (what data you ingest)
Step 2: Import "Threat Intel" layer (techniques targeting your sector)
Step 3: Import "Active Detections" layer (current SIEM rules)

Analysis:
- Enable ALL layers → See total addressable coverage
- Disable "Active Detections" → See gaps where you should build rules
- Yellow/Red areas = High priority detection engineering targets
```

#### Workflow 3: Platform Coverage Validation

The v18 schema automatically handles platform specificity through Analytics and Data Components:

- AN1336 = Windows Event Logs
- AN1338 = AWS CloudTrail
- AN1339 = EDR Process Monitoring

If your layer activates AN1336 but not AN1338, the visualization shows:
- ✅ T1110 covered on Windows
- ❌ T1110 NOT covered on Cloud

This prevents the "false positive coverage" problem where you think T1110 is covered, but it's only covered on-prem.

**Note:** Full automation of SIEM → v18 mapping requires rule metadata enrichment in your CI/CD pipeline. Add `mitre_detection_strategy` and `mitre_analytic` tags to your detection rules.

## File Format Examples

See included example files:

- `example_sentinel_detections.json` - Production SIEM rules

## Color Coding

The sunburst visualization uses a 10-step gradient for coverage percentage:

- 🔴 0-30%: Dark Red → Red (Critical gaps)
- 🟠 30-50%: Orange (Needs improvement)
- 🟡 50-70%: Yellow (Moderate coverage)
- 🟢 70-100%: Green (Good coverage)

## Technical Architecture

### Data Flow

```
MITRE STIX v18 (GitHub)
    ↓
Parse to Hierarchy (Tactic→Technique→Strategy→Analytic→Component)
    ↓
Import Detection Layers (JSON)
    ↓
Map Detections to Analytics/Data Components
    ↓
Calculate Coverage (Union of all enabled layers)
    ↓
Render Sunburst with Coverage Gradient
```

### Storage

- **MITRE Data:** Fetched from GitHub on page load
- **Layers:** Stored in browser `localStorage`
- **Persistence:** Automatic save on layer changes

### Browser Compatibility

- Chrome/Edge: ✅ Recommended
- Firefox: ✅ Full support
- Safari: ✅ Supported (14+)

## Advanced Features

### Manual Toggle Mode

When NO layers are enabled, right-click on any segment to manually toggle Data Components. This is useful for:

- Exploring "what-if" scenarios
- Testing coverage hypotheticals
- Educational demonstrations

**Note:** Manual mode is disabled when layers are active to prevent conflicts.

### Layer Source Tracking

Hover over any Data Component to see which layer(s) activated it:

```
Active in: Sentinel Coverage, EDR Visibility
```

This helps understand *why* a component is covered and prevents duplicate rule deployments.

### Collapsible Schema Reference

Click "Schema Reference" in the sidebar to view the full v18 detection layer format inline.

## Troubleshooting

### "No layers imported yet"

- Ensure your JSON file matches the v18 schema
- Check browser console for validation errors
- Verify `detections` array exists and is not empty

### "Layer imported but no coverage change"

**This is the most common issue.** The layer imported successfully but didn't activate any Data Components. Here's how to debug:

1. **Open Browser Console** (F12 → Console tab)

2. **Check what IDs are available:**
   ```javascript
   // List all available Analytics in the v18 data
   listAnalytics()
   
   // List all available Data Components
   listDataComponents()
   
   // Find Analytics for a specific technique
   findAnalyticsByTechnique("T1110")
   ```

3. **Verify your layer uses correct IDs:**
   - Analytics IDs must match those shown by `listAnalytics()`
   - Data Component IDs must match those shown by `listDataComponents()`
   - The v18 STIX data may not include all theoretical IDs yet

4. **Common ID issues:**
   - **Invented IDs**: The example files use hypothetical IDs (AN1336, AN1338, etc.) that may not exist in the actual v18 STIX data
   - **Incorrect format**: Ensure IDs follow the pattern `ANxxxx`, `DETxxxx`, `DSxxxx`
   - **Case sensitivity**: IDs are case-sensitive

5. **Workaround - Direct Data Component activation:**
   
   If Analytics don't exist yet in v18, you can activate Data Components directly:
   
   ```json
   {
     "ruleName": "My Detection",
     "ruleId": "RULE-001",
     "tactics": ["credential-access"],
     "techniques": ["T1110"],
     "subtechniques": [],
     "detectionStrategies": [],
     "analytics": [],
     "dataComponents": ["DS0028"],
     "tables": ["SecurityEvent"],
     "score": 1.0,
     "enabled": true
   }
   ```
   
   Use `listDataComponents()` in console to find valid DS IDs, then reference them directly.

6. **Check the debug logs:**
   
   After importing, the console will show:
   ```
   Processing detection: High Volume Auth Failures
     Analytics to activate: AN1336, AN1338
     ✗ Failed to find Analytic with ID: AN1336
     ✗ Failed to find Analytic with ID: AN1338
   ```
   
   This tells you exactly which IDs don't exist in the STIX data.

### Creating Production Layers

Since the v18 Analytics release is very recent (October 2025), the STIX data may still be evolving. For production use:

1. **Use the console helpers** to discover actual IDs after the page loads
2. **Reference Data Components directly** (DSxxxx IDs) - these are stable
3. **Update your SIEM export script** to use the IDs discovered via the helpers
4. **Use the test_layer_simple.json** as a template - it activates Data Components without requiring Analytics

### Example: Discovering Real IDs

```javascript
// Step 1: Load the page and wait for MITRE data
// Step 2: Open console and run:

listDataComponents()
// Output: [DS0001, DS0002, DS0003, ...]

// Step 3: Find components related to authentication
window.debugAvailableDataComponents.filter(id => {
  // Would need to cross-reference with MITRE website
  // For now, DS0028 is typically "Logon Session"
  return id.includes('DS002')
})

// Step 4: Use those IDs in your layer JSON
```

### Performance Issues

- Limit layers to ~10 maximum
- Consider splitting large rulesets into focused layers

## Future Enhancements (Roadmap)

- [ ] **Robustness Scoring:** Integrate DRAPE Index
- [ ] **Export to Navigator:** Convert v18 layers to Navigator v4.5 format
- [ ] **Layer Intersection Mode:** Show only gaps (Visibility ∩ Threat - Detections)
- [ ] **CI/CD Integration:** Webhook endpoint for automated layer updates
- [ ] **Diff Visualization:** Compare layer versions over time
- [ ] **General UX:** Introduce seperate tabs for layers and visibility.

## References

- MITRE ATT&CK v18 Release Notes: https://attack.mitre.org/resources/updates/updates-october-2025/
- Detection Strategies: https://attack.mitre.org/detectionstrategies/
- Analytics: https://attack.mitre.org/analytics/

## License

This tool uses MITRE ATT&CK data licensed under Apache 2.0.

---

**Built for detection engineers, by detection engineers.** 
