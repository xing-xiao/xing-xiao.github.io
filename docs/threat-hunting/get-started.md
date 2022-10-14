# 一文读懂威胁狩猎(Threat Hunting)

## 前言

在进入正文之前，我们先来介绍

## Threat Hunting简介

### 什么是Threat Hunting

首先，我们需要先明确什么是Threat Hunting， 在给出我自己的答案之前，

我们先来看看行业内其他机构是如何理解Threat Hunting，如在SANS中有多篇文章给出了定义：

*Threat hunting is the proactive approach of searching and finding threats within an organization’s network that may go undetected for a long time due to weaknesses in traditional reactive detection systems and techniques.* [[5](https://www.sans.org/white-papers/39610/)]

和

*Threat hunting uses new information on previously collected data to find signs of compromise evading detection.* [[6](https://www.sans.org/white-papers/39025/)]


这些定义的前提都建立在威胁检测(Detect)无法完全发现所有的恶意入侵，造成这样的原因包括：检测技术存在缺陷、不断出现的新型攻击手法、检测的覆盖面不足等等原因。而Threat Hunting座位一种主动的周期性活动，用于发现威胁检测中未发现的威胁，IPDRR安全框架中用于弥补识别(IDENTIFY)、保护(PROTECT)和检测(DETECT)的不足。我们将会在下一小节**Threat Hunting在安全防御框架中的作用**中详细介绍威胁狩猎的作用，以及在安全防御框架中它与其他安全活动之间的关系。

Chris Brenton的文章[what is threat hunting and why is it so important [7]](https://www.activecountermeasures.com/what-is-threat-hunting-and-why-is-it-so-important-video-blog/)中也讲了对Threat Hunting的观点，主要内容包括：Threat Hunting是一个active/proactive活动，对象是所在组织的任何相关环境(原文中用词是everything)，内容是发现任何失陷的信号(signs of being compromised),而输出是是否失陷的评估。

在最新的BlackHat USA(2022年8月)会议中，来自IBM X-Force的专家John Dwyer和Neil Wyler分享了议题[Open Threat Hunting Framework [8]](https://www.blackhat.com/us-22/briefings/schedule/#the-open-threat-hunting-framework-enabling-organizations-to-build-operationalize-and-scale-threat-hunting-26702)，体系化的阐述了Threat Hunting的体系框架。这篇文章在2.3小节中则建议由开展Threat Hunting活动的组织在内部自己定义Threat Hunting以及相关活动，而定义的内容应该包括：1.hunting不针对已经能检出的威胁；2.是应该专项的、周期性开展的活动；3.活动是建立在“假设”的基础上。在这个文章中也给出了多家安全厂商对Theat hunting的定义，包括：

- hreat Hunting is a dedicated, continuous, hypothesis-based search methodology to reduce the time to detect adversaries operating within an environment that have yet to be detected.
- Threat hunting is the practice of proactively searching for cyber threats that are lurking undetected in a network. - CrowdStrike[9]
- Cyber threat hunting is a proactive security search through networks, endpoints, and datasets to hunt malicious, suspicious, or risky activities that have evaded detection by existing tools. - Trellix[10]
- Threat hunting is the practice of searching for cyber threats that might otherwise remain undetected in your network. - CheckPoint[11]

综上，我们总结一下Threat Hunting，需要包含一下内容：

1. Threat Hunting是基于环境已经失陷的假设，而且这种失陷未被现有的安全防御体系检测到，
2. 由安全运营相关的人员主动的开展分析、搜索活动，
3. 活动应该是周期性开展，
4. 活动应该由明确的产出，包括且明确的是否失陷的结论、失陷发现的分析手段(以规则、代码等形式展现)、对已有防御体系中不足的改进建议、缺失的检测数据、总结的新增检测规则等。
5. 目标是解决信息安全防御体系的痛苦金字塔(Pyramid of Pain)中顶端的痛点[13]，提高攻击者入侵的门槛。

### TH在SOC中的位置

传统SOC：SIEM检测出恶意行为并告警，SOC运营人员由这些告警触发去开展安全调查。而攻击者也在不断提高自己的技巧以规避检测。
consider threat hunting to be an essential step in detecting adversaries and forms part of a complete security program. [What Is Threat Hunting and Why Is It so Important? – Chris Brenton](https://www.activecountermeasures.com/what-is-threat-hunting-and-why-is-it-so-important-video-blog/)

Incident Response 生命周期：[NIST.SP.800-61r2](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

[NIST IPDRR Framework](https://www.nist.gov/cyberframework)



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


实验环境

- https://github.com/clong/DetectionLab

## Reference

1. [SANS 2022 Threat Hunting Survey - Hunting for a Standard Methodology for Threat Hunting Teams](https://www.youtube.com/watch?v=n29whvCuwhc)
2. [Twitter ThreatHuntProj](https://twitter.com/ThreatHuntProj)
3. [https://www.threathunting.net/](https://www.threathunting.net/)
4. [https://resources.infosecinstitute.com/topics/threat-hunting/](https://resources.infosecinstitute.com/topics/threat-hunting/)
5. [Applying the Scientific Method to Threat Hunting - Jeremy Kerwin (thalesgroup.com.au) - SANS 2020](https://www.sans.org/white-papers/39610/)
6. [Building and Maturing Your Threat Hunting Program - David Szili (CTO @ Alzette Information Security) - SANS 2019](https://www.sans.org/white-papers/39025/)
7. [What Is Threat Hunting and Why Is It so Important? – Video Blog](https://www.activecountermeasures.com/what-is-threat-hunting-and-why-is-it-so-important-video-blog/)
8. [The Open Threat Hunting Framework: Enabling Organizations to Build, Operationalize, and Scale Threat Hunting - BlackHat USA 2022](https://www.blackhat.com/us-22/briefings/schedule/#the-open-threat-hunting-framework-enabling-organizations-to-build-operationalize-and-scale-threat-hunting-26702)
9. [https://www.crowdstrike.com/cybersecurity-101/threat-hunting/](https://www.crowdstrike.com/cybersecurity-101/threat-hunting/)
10. [https://www.trellix.com/en-us/security-awareness/operations/what-is-cyber-threat-hunting.html](https://www.trellix.com/en-us/security-awareness/operations/what-is-cyber-threat-hunting.html)
11. [https://www.checkpoint.com/cyber-hub/cloud-security/what-is-threat-hunting/](https://www.checkpoint.com/cyber-hub/cloud-security/what-is-threat-hunting/)
12. [Threat Hunting Tutorial: Introduction](https://www.youtube.com/watch?v=qrZsc5IkchI)
13. [A Framework for Cyber Threat Hunting Part 1: The Pyramid of Pain - Sqrrl Team](https://www.threathunting.net/files/A%20Framework%20for%20Cyber%20Threat%20Hunting%20Part%201_%20The%20Pyramid%20of%20Pain%20_%20Sqrrl.pdf)
14. 