# Beyond the Bingo Card: Engineering Detection Depth with MITRE ATT&CK v18

I'm sure many of you are more than aware about the fundemental challenges we have in our industry regarding in how we measure and report detection capabilities. For years, the MITRE ATT&CK Matrix has been our "North Star" and it's certainly served us well providing much needed organisation and clarity regarding how to prioritise gaps in our detection coverage. However the total focus on this has led to an oversimplification: the reduction of complex, multi-dimensional attack surfaces into a two-dimensional "Bingo Card".

In many organisations, the goal has become the visual saturation of the matrix—turning cells green to signal "coverage". However a single or even many detections against a Mitre Technique or even subtechnique measures at far too high a level to measure effectiveness of coverage against all the different ways that Technique may be expressed across different methods or technologies. 

* **Dimensionality Loss**: A single green box for "Brute Force" (T1110) implies safety, but a rule detecting Windows Event ID 4625 provides zero coverage for the same technique executed against AWS CloudTrail or a Linux shadow file.


* **Wrongly incentivizing** Another issue with this issue of the current way we might express coverage through The "Bingo Card" can incentivise "fragile" detections for example a simple string or hash matches—because they "turn the box green" quickly but are easily bypassed by a single byte change from an attacker.


* **Binary Metrics**: It fails to capture the **robustness** of a detection. A "Level 1" IOC detection (based on a rotating IP) is visually identical to a "Level 5" "Standard Deviation Anomaly" detection (based on statistical anomalies), yet their operational value is worlds apart.

* **Detection Efficacy** Finally we are failing to capture the ongoing efficacy of a detection, it's TP percentage or it's effectivenes when tested with BAS or Adversay Simulation tooling.

## Potential solutions via MITRE ATT&CK v18

The release of MITRE ATT&CK v18 (October 2025) transforms the framework from a flat "Dictionary" into a "Relational Database" of defense. By introducing **Detection Strategies (DETxxxx)** and **Analytics (ANxxxx)**, v18 provides the primitives needed to map the true depth of coverage.

Mulling this over Christmas I found this article back from 2019 [Visualising Att&ck](https://medium.com/mitre-attack/visualizing-attack-f5e1766b42a6) Which I took as inspiration to create an interactive Sunburst visualisation of the new Mitre v18 framework which people could upload their own layers to simiarly to the Mitre Navigator tool. So with some obvious heavy lifting by LLM's I've built the **MITRE ATT&CK v18 Detection Depth Map**, a sunburst-based visualization that fully embraces the v18 hierarchy:

> **Tactic → Technique → Detection Strategy → Analytic → Data Component**

### 1. Detection Strategies (The "Logic Layer")

Strategies represent high-level defensive approaches independent of specific vendors. Instead of saying "We cover Brute Force," we can now say "We cover **Volume-based detection**, but we are exposed to **Distributed Low-and-Slow** strategies".

### 2. Analytics (The "Implementation Layer")

Analytics are the platform-specific rules (Windows, Linux, Cloud) that link strategies to real-world telemetry. This forces the explicit declaration of the platform dimension—you cannot have a "generic" analytic; it must be defined by its environment.

### 3. Data Components (The "Visibility Layer")

Data Components (DSxxxx) act as the **Bill of Materials** for detection. Data Components identify the specific properties/values relevant to detecting a given ATT&CK technique or sub-technique.

## Final Thoughts

This POC is very much just that, there are plenty of improvements I'm already planning to add myself importantly Implementation of the "Detection Efficacy" Score which is present in the json schema but not yet shown visually in the tool.
Also admittedly the UI needs a lot of work but hopefully as an idea for how to more accurately represent coverage using the new Mitre Att&ck v18 Schema.
