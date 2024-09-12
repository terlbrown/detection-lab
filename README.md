# Battle-Ready: How Detection Labs Shape Elite Cyber Defenders

<img src="https://github.com/terlbrown/detection-lab/blob/main/images/battle-ready-detection-lab.webp" width=50% height=50%>

In the relentless world of cybersecurity, the blue team stands as the frontline defense, tasked with protecting our organizations from ever-evolving threats. Yet amidst the daily grind of alerts, investigations, and routine operations, there’s a silent challenge that often goes unspoken: the struggle to maintain and sharpen the very skills that could mean the difference between a successful defense and a devastating breach.

Blue teamers are expected to be agile, innovative, and precise, but the reality is that the routine nature of daily tasks often leaves little room for the critical practice needed to stay ahead of adversaries. The skills required in the most crucial moments—when the stakes are highest—are often those that are least exercised in the daily flow of operations. This disconnect can leave even the most seasoned defenders vulnerable, not because of a lack of knowledge, but due to a lack of opportunity to refine and apply that knowledge in meaningful ways.

Today, we’re going to address this challenge head-on. We’ll explore how detection labs can become the proving ground where blue teamers can continuously hone their skills, ensuring they are not just prepared but truly battle-ready.

## Pivoting Lab Overview

This lab is designed to teach cybersecurity professionals the important skill of **pivoting** in penetration testing. Pivoting is a technique used by attackers to move laterally through a network after compromising a machine. Once they gain initial access to a system, attackers use it as a stepping stone to exploit other systems within the internal network. This technique is vital for security defenders to understand so they can identify and mitigate such attacks in real-world scenarios.

### Purpose:
The main objective of this lab is to provide a controlled environment where blue team members can practice techniques for identifying and defending against lateral movement attacks. The lab simulates a scenario in which an attacker compromises an initial system and then pivots through the network to gain access to more critical resources.

### Setup:
The lab infrastructure is deployed on AWS using **Terraform** for automated provisioning. The environment includes:
- **Attacker Instance**: A Kali Linux machine within a private subnet that simulates an adversary attempting to penetrate the network.
- **Target Instances**: Two vulnerable target machines (one public-facing and one private), simulating services that might be exploited by the attacker.

The networking architecture is set up to reflect realistic segmentation, including **public** and **private subnets**, an **Internet Gateway**, and **NAT Gateway**, which provide secure communication paths between resources while maintaining isolation where required.

### Learning Objectives:
Participants will learn to:
- Leverage tools and techniques to exploit an initial foothold within a network.
- Use compromised machines to access other internal resources that would not be directly accessible from the internet.
- Detect and respond to lateral movement within an internal network.
- Practice real-world security defense techniques by using detection tools to identify pivoting attacks.

### Simulation:
The lab culminates in a penetration testing exercise that challenges participants to:
1. Gain access to the first target machine.
2. **Pivot** to other internal systems that are otherwise unreachable directly from the attacker machine.
3. Complete the task by obtaining critical data (flags) from compromised systems to prove the success of the lateral movement.

This lab allows defenders to train and perfect their response to these types of attacks in a cloud environment, ensuring they are prepared to handle similar scenarios in production.


### Quick Links:
For detailed documentation and instructions, visit the following:
- [AWS Account Setup Instructions](./docs/setup.md)
- [Pivoting Lab Walkthrough](./docs/README.md)
- [Terraform Configuration Guide](./docs/terraform.md)

---
