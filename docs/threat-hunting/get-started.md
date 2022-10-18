# 一文读懂威胁狩猎(Threat Hunting)

## 前言

很早就想针对威胁狩猎(Threat Hunting)来写一篇详细文章介绍体系，作为安全运营活动中非常重要的一环，国外已经由很多文章和体系的介绍，8月份的BlactHat USA专题演讲OTHF，但是国内介绍的很少。

## Threat Hunting简介

### 什么是Threat Hunting

首先，我们需要先明确什么是Threat Hunting， 在给出我自己的答案之前，我们先来看看行业内其他机构是如何理解Threat Hunting，如在SANS中有多篇文章给出了Threat Hunting的定义：

*Threat hunting is the proactive approach of searching and finding threats within an organization’s network that may go undetected for a long time due to weaknesses in traditional reactive detection systems and techniques.* [[5](https://www.sans.org/white-papers/39610/)]

和

*Threat hunting uses new information on previously collected data to find signs of compromise evading detection.* [[6](https://www.sans.org/white-papers/39025/)]


这些定义的前提都建立在威胁检测(Detect)无法完全发现所有的恶意入侵，造成这样的可能原因包括：检测技术存在缺陷、不断出现的新型攻击手法、检测的覆盖面不足等等。而Threat Hunting作为一种主动的周期性活动，可以用于发现威胁检测过程中因为上述原因而未被发现的威胁。在NIST的IPDRR安全框架中，Threat Hunting是用于弥补识别(IDENTIFY)、保护(PROTECT)和检测(DETECT)的不足，这一点我们会在下一小节**Threat Hunting在安全防御框架中的位置**中详细的展开介绍，并系统讲解安全防御框架中Threat Huting与其他安全活动之间的关系。

Active Countermeasures的COO、SANS的讲师Chris Brenton在Youtube上发表了一系列的教程[视频[14]](https://www.youtube.com/watch?v=lt1ld62Fids)来讲述Threat Hunting活动(ps. 视频一开始的妹子Shelby带上粉色猫耳朵耳麦很萌，比他在AC官网的照片要好看的多)。Chris Brenton在2020年的文章[what is threat hunting and why is it so important [7]](https://www.activecountermeasures.com/what-is-threat-hunting-and-why-is-it-so-important-video-blog/)中简要讲了对Threat Hunting的理解以及重要性，观点主要包括：Threat Hunting是一个active/proactive活动，对象是所在组织的任何相关环境(原文中用词是everything)，内容是发现任何失陷的信号(signs of being compromised),而输出是是否失陷的评估。

在最近的BlackHat USA(2022年8月)会议中，来自IBM X-Force的专家John Dwyer和Neil Wyler分享了议题[Open Threat Hunting Framework (OTHF)[8]](https://www.blackhat.com/us-22/briefings/schedule/#the-open-threat-hunting-framework-enabling-organizations-to-build-operationalize-and-scale-threat-hunting-26702)，体系化的阐述了Threat Hunting活动框架。这篇文章在2.3小节中则是建议由开展Threat Hunting活动的组织在内部自己定义Threat Hunting以及相关活动，而定义的内容应该包括：1.hunting不针对已经能检出的威胁；2.是应该专项的、周期性开展的活动；3.活动是建立在“假设”的基础上。

在OTHF文章中也给出了多家安全厂商对Theat hunting的定义，内容大同小异，包括：

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

### Threat Hunting在安全防御框架中的位置

Threat Hunting是安全防御框架和安全运营活动中重要的一环，在本章节我们来简单的介绍安全运营活动的框架，以及Threat Hunting在整个安全运营活动中的位置和作用，这样能够更好的帮助我们理解为何和如何开展Threat Hunting。同时我也在计划针对安全运营(SOC)单独写一盘详细的文章，请关注我的博客或者公众号。

NIST 信息安全事件处置指南[NIST.SP.800-61r2 [18]](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)中，将事件响应(Incident Response)的生命周期分为`准备`、`检测和分析`、`阻止、消除和恢复`、`后事件处置活动`四个阶段。在[The Foundations of Threat Hunting [17]](https://www.amazon.com/Foundations-Threat-Hunting-Organize-effective/dp/180324299X)这本书中，列举了IR生命周期每个阶段中会包含的安全运营活动列表，其中Threat Hunting归纳到了事件响应生命周期的`检测和分析`阶段，他将Threat Huting作为威胁检出的一项手段，也就是做为使用规则进行威胁发现和告警聚合的一种补充，主要目标还是尽可能的增加威胁的检出率，如下图所示。

![Incident Response 生命周期](image/incident-response-lifecycle.png)

而在[NIST的安全框架IPDRR[16]](https://www.nist.gov/cyberframework)中，将将安全活动分为`Identify`、`Protect`、`Detect`、`Respond`、`Recover`的闭环活动，覆盖了针对企业资产的完整信息安全保护的周期，简称IPDRR框架

![NIST Cyber security Framwork v1.1](image/framework_functions_wheel.png)

IPDRR每一个阶段的细分项目内容如下：

![NIST IPDRR Category Unique Identifier](image/IPDRR-Category.png)

其中`DE.CM`阶段表示`Security Continuous Monitoring: The information system and assets are monitored to identify cybersecurity events and verfy the effectiveness of protective measures`，他在`DE.DP`之前，`DE.DP`阶段主要内容包括`Detection processes and procedures are maintained and tested to ensure awareness of anomalous events`。而Threat Hunting涉及了DE.CM-1、DE.CM-2、DE.CM-3、DE.CM-6、DE.CM-7，用于发现环境和组织中潜在的恶意威胁，同时Threat Hunting的结果也会反馈DE.DP-5 `Detection processes are continuously improved`。

传统SOC：SIEM检测出恶意行为并告警，SOC运营人员由这些告警触发去开展安全调查。而攻击者也在不断提高自己的技巧以规避检测。
consider threat hunting to be an essential step in detecting adversaries and forms part of a complete security program. [What Is Threat Hunting and Why Is It so Important? – Chris Brenton](https://www.activecountermeasures.com/what-is-threat-hunting-and-why-is-it-so-important-video-blog/)

![Daily defenses versus hunt](image/daily-defenses-versus-hunts.png)



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
[Stay ahead of the game: automate your threat hunting workflows](https://www.youtube.com/watch?v=UEwplUM5GlU)  6:20


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
14. [Cyber Threat Hunting Level 1](https://www.youtube.com/watch?v=lt1ld62Fids)
15. [Active Countermeasures Youtube 频道](https://www.youtube.com/c/ActiveCountermeasures)
16. [NIST网络安全框架](https://www.nist.gov/cyberframework)
17. [The Foundations of Threat Hunting](https://www.amazon.com/Foundations-Threat-Hunting-Organize-effective/dp/180324299X)
18. [NIST.SP.800-61r2](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)