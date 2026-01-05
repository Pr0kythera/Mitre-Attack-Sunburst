# Beyond the Bingo Card: Engineering Detection Depth with MITRE ATT&CK v18

I'm sure many of you are more than aware about the fundamental challenges we have in our industry regarding how we measure and report detection capabilities. For years, and for good reason, the MITRE ATT&CK Matrix has been our "North Star" and it's certainly served us well providing much needed organisation and clarity regarding how to prioritise gaps in our detection coverage. However the focus on this has led us into an oversimplification. The reduction of complex, multi-dimensional attack surfaces into a two-dimensional ["Bingo Card"](https://www.forrester.com/blogs/the-mitre-attck-framework-is-not-a-bingo-card/). That being said I am by no means deriding the practice of measuring coverage and identifying gaps via Mitre Navigator layers. Especially since last year we gave a presentation on exactly this methodology at Bsides Birmingham!

In many organisations the goal has become the visual saturation of the matrix, turning cells green to signal "coverage". However a single or even many detections against a Mitre Technique or subtechnique in reality measures at a far too high a level to gauge real world effectiveness. Tracking coverage at the Technique or even Sub-Technique level doesn't comprehensively convey all the different ways that Technique may be expressed across different methods or technologies. Whilst procedures offer a more precise invocation of a technique they have historically not really been employed as mappings likely because they're usually specific historic examples of how Threat Actor X utilised a technique rather than a more generalised approach which could be applied across detections.

To highlight some key shortcomings in the current approach:

* **Dimensionality Loss**: A single green box for "Brute Force" (T1110) implies safety. A rule detecting Windows Event ID 4625 provides zero coverage for the same technique executed against AWS CloudTrail or a Linux shadow file. An attacker may unknowingly target services which are not covered by detections using techniques we have "great" coverage of.

* **Wrongly incentivizing** Another issue with this issue of the current way we might express coverage through The "Bingo Card" can incentivise "fragile" detections for example a simple string or hash matches—because they "turn the box green" quickly but are easily bypassed by a single byte change from an attacker. Whilst engineers will thoroughly want to reject this premise to achieve X% coverage by Y date as a teams KPI may inevitably lead to cut corners. 

* **Binary Metrics**: It fails to capture the **robustness** and **efficacy** of a detection. A fragile IOC detection (based on a rotating IP) is visually identical to a more robust "Standard Deviation Anomaly" detection or fancy Machine Learning Model (based on statistical anomalies), yet their operational value is often worlds apart. Whilst I don't extensively cover any proposed solutions in this article there has been some great writeups covering this topic like [DRAPE](https://detect.fyi/introducing-the-drape-index-how-to-measure-in-success-in-a-threat-detection-practice-154fd977f731)


## Potential solutions via MITRE ATT&CK v18

I had been eagerly anticipating the release of MITRE ATT&CK v18 (October 2025) after reading this [blog post](https://medium.com/mitre-attack/smarter-detection-strategies-in-attack-7e6738fec31f) in the summer by Lex Crumpton. It appeared to be the missing concepts needed to bridge the gap and transform the framework from a flat "Dictionary" into more of a "Relational Database" of defense. By introducing **Detection Strategies (DETxxxx)** and **Analytics (ANxxxx)**v18 provides the primitives needed to more accurately represent the depth of coverage.

Mulling this over Christmas I found this article back from 2019 [Visualising Att&ck](https://medium.com/mitre-attack/visualizing-attack-f5e1766b42a6) Wwich I took as inspiration to create an interactive Sunburst visualisation of the new Mitre v18 framework. The goal is a tool to allow people to upload their own layers similar to the Mitre Navigator tool. So with some obvious heavy lifting by LLM's I've built the **MITRE ATT&CK v18 Detection Depth Map**, a sunburst-based visualization that fully embraces the v18 hierarchy:

As a pre-amble I will be the first to concede this is practically useless unless Detection Engineering teams tag their detections with all of the new primitives to accurately represent their coverage on this tool. Currently I'd be surprised if any team is doing this especially as there aren't many tools to visualise the mapping. However this would have been the case only a few years ago with Mitre Techniques not being represented on Detection Rules. So whilst this proposed solution is somewhat academic in nature and a lot of heavy lifting would be required to get value from it my only hope is to highlight some shortcomings and propose potential tangible solutions. Caveat emptor and all that..

The new Primitives introduced: 

> **Tactic → Technique → Detection Strategy → Analytic → Data Component**

### 1. Detection Strategies (The "Logic Layer")

Strategies represent high-level defensive approaches independent of specific vendors. Instead of saying "We cover Brute Force," we can now say "We cover **Volume-based detection**, but we are exposed to **Distributed Low-and-Slow** strategies".

### 2. Analytics (The "Implementation Layer")

Analytics are the platform-specific rules (Windows, Linux, Cloud) that link strategies to real-world telemetry. This forces the explicit declaration of the platform dimension—you cannot have a "generic" analytic; it must be defined by its environment.

### 3. Data Components (The "Visibility Layer")

Data Components (DSxxxx) act as the **Bill of Materials** for detection. Data Components identify the specific properties/values relevant to detecting a given ATT&CK technique or sub-technique.

## Final Thoughts

Even at the Data Component layer due to the incomplete nature Mitre Att&ck has, we're still going to not get a perfect representation of your environment. This would likely require a method to build your own entire Organization Matrix which has every SAAS application and Software listed. But rather than letting perfect be the enemy of the good, I feel like we've been able to highlight some of the known gaps in traditional Detection mapping to Techniques and Tactics and how we can set our sights on new primitives offered by the V18 update. 

## The Tool POC

https://pr0kythera.github.io/Mitre-Attack-Sunburst/attack_v18_detection_layers.html

Rather than just discuss theory I wanted to construct this tool into a flawed but hopefully illustrative vision of the goal. From the traditional 500 or so subtechniques we have over 5000 components. It has basic functionality to enable data components which will light up green and the ability to import your own layers. This was hastily put together and not designed for actual enterprise use in its current iteration.

This POC is very much just that, there are plenty of improvements I'm already planning to add. Also admittedly the UI needs **a lot** of work.. But hopefully as an idea and inspiration for how to better represent coverage using the new Mitre Att&ck v18 Schema it has some use

Some future improvements, any other ideas are welcome:

- Implementation of the "Detection Efficacy" Score which is present in the json schema but not yet shown visually in the tool
- Ability to "diff" two layers to show coverage change between them.
- UI overhaul
- Potentially shifting subtechniques in a dimension below techniques rather than alongside them. Taxonomically they should sit below but it might create too many dimensions to have to sift up and down through? 
