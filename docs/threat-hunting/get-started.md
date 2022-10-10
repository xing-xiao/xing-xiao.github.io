# Threat Hunting

https://www.threathunting.net/

## Introduction

### TH在SOC中的位置

传统SOC：SIEM检测出恶意行为并告警，SOC运营人员由这些告警触发去开展安全调查。而攻击者也在不断提高自己的技巧以规避检测。
consider threat hunting to be an essential step in detecting adversaries and forms part of a complete security program. [What Is Threat Hunting and Why Is It so Important? – Chris Brenton](https://www.activecountermeasures.com/what-is-threat-hunting-and-why-is-it-so-important-video-blog/)

Incident Response 生命周期：[NIST.SP.800-61r2](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

[NIST IPDRR Framework](https://www.nist.gov/cyberframework)

### TH的定义

定义： it is the proactive approach of searching and finding threats within an organization’s network that may go undetected for a long time due to weaknesses in traditional reactive detection systems and techniques.[Applying the Scientific Method to Threat Hunting](https://www.sans.org/white-papers/39610/)

1. Search the network for signs of compromise
2. Active/Proactive, not reactive
3. Should include all device-servers, desktops, network hardware, IIOT, BYOD
4. Output is a compromise assessment

[Threat Hunting Tutorial: Introduction](https://www.youtube.com/watch?v=qrZsc5IkchI)

是什么(What) -> 为什要做(Why) -> 从哪里开始(Where) -> 如何去做(How) -> 什么角色做(Who) -> 产出(Output)

### TH为什么重要或者必须的

team level benifits
• Turns unknown risks into known risks and allows them to be managed effectively
• Identifies adversarial activities that made it through existing defenses
• Provides an increased understanding of what threats current defenses have visibility into and where those defenses could be lacking
• Increases understanding of the enterprise for all personnel involved
• Validates/develops a documented network baseline and map
• Provides insight into potential system and network misconfigurations
• Identifies gaps in logging and network visibility

high level begifits
• Improves adherence to legal and regulatory requirements
• Aides in risk management decisions before or after major network reconfigurations, such as mergers with other organizations
• Validates threat intelligence reporting specific to the organization and the threat actors that are targeting them
• Can be utilized as a proof point for any investment adjustments into specific network security areas
• Re-enforces stakeholders' trust in the confidentiality, integrity, and availability of the network


## Movitation & preparation

where to start

Types of threat hunts - TH Category quad chart
• Can it be narrowed down to a specific threat actor?
• Can it be narrowed down to specific attack capabilities or techniques?
• Can it be narrowed down to a level of sophistication?


Intelligence驱动
TTP驱动
Anomaly驱动
https://www.youtube.com/watch?v=UEwplUM5GlU  6:20


Scopes of threat hunts


## Team Construct


## Methodologies

enables team to have repeatable process
consistently aligns its efforts
get every one on the same page

## Process

数据：[Security Datasets project](https://securitydatasets.com/)

Threat Hunting Tool:
- 微软 [MSTIC Jupyter and Python Security Tools](https://github.com/microsoft/msticpy)
- [Threat Hunting Project hunt tool](https://github.com/ThreatHuntingProject/hunter)
- https://github.com/reprise99/Sentinel-Queries

Threat Hunting Queries:
- [Sentinel](https://github.com/DanielChronlund/DCSecurityOperations) 和 [Blog](https://danielchronlund.com/2022/10/03/sentinel-hunting-query-pack-dcsecurityoperations/)
- [Bert-Jan](https://github.com/Bert-JanP/Hunting-Queries-Detection-Rules)

使用假设，然后通过SIEM中数据来验证这个假设是否正确

作用：补充现有的工具和方法、减少攻击的检测时间、识别网络中检测能力的差距，帮助保持领先于快速变化的威胁并提供快速响应这些威胁的机制。

方法：科学方法是通过实验检验假设以回答问题的过程。在医学、生物学、化学和物理学等领域的科学进步中，它的使用可以追溯到数百年前。
[Steps of the Scientific Method](https://www.sciencebuddies.org/science-fair-projects/science-fair/steps-of-the-scientific-method)


建立可重复的、标准化、可测试的、可度量的TH方法论，

searching something not currently being detected,  searching, not necessarily finding

isn't evil does not mean it isn't interesting


Threat Hunting和Adversary Simulation区别
TH 和 SOAR关系  Automating Everything
TH 和 IR关系
TH 由 TI驱动
TH 和安全巡检，覆盖面不同，目的不同

## Reference

- [SANS 2022 Threat Hunting Survey - Hunting for a Standard Methodology for Threat Hunting Teams](https://www.youtube.com/watch?v=n29whvCuwhc)
- https://twitter.com/ThreatHuntProj