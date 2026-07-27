---
title: "Results from Secret Loyalties Hackathon"
date: 2026-07-27
author: "Matt Allan"
layout: post
tags:
  - "AI Safety"
---

## Overview

Recently, the mission of AI safety has really spoken to me. As AI becomes more integrated into our lives and workspaces, the need for proactive security and alignment is more crucial than ever. As this challenge has grown clearer, I've wanted to apply my skill set to be a part of the solution. 

Over the weekend, July 24th-26th, I took part in a Apart Research's Secret Loyalty Hackathon. Apart is on a mission to enable researchers the ability publish original research through sprints (like in this case), fellowships and funding assistance. Please check out their website [here](https://apartresearch.com/).

### Objectives

The Secret Loyalties Hackathon was inspired by the paper _"AIs with Secret Loyalties are a Serious but Addressable Threat"_ by Joe Kwon, Alfie Lamerton, et al. (2026). In the paper, the authors argue that an AI model can be trained or conditioned to possess hidden biases—a "secret loyalty"—that advance the specific interests of an undisclosed principal (such as a nation-state, executive, or corporate competitor).

Unlike traditional software backdoors triggered by specific keywords, a secretly loyal model can continuously scan its interactions to subtly tilt decision-making, code generation, or policy advice in favor of its principal without users or auditors noticing.

To address this threat, participants were asked to tackle one of five research tracks:

1. **Model Organisms:** Build reproducible model organisms of secret loyalties spanning the activation-breadth $\times$ action-breadth space.
2. **Evaluation & Auditing (Selected):** Evaluate whether existing backdoor and auditing defenses transfer to secret loyalties across graded affordance levels.
3. **Red-Teaming Attack Pathways:** Test pathways like data poisoning, multi-stage priming, and reasoning-trace poisoning under controlled conditions.
4. **Detection & Defense:** Detect or remediate loyalties post-hoc using interpretability probes or last-mover training strategies.
5. **Threat Modeling & Governance:** Theorize threat models, leverage analogies from cybersecurity/insider threat frameworks, and analyze scaling trends.

I selected **Track 2: Evaluation**.

### Methodology

Apart Research and Formation Research provided participants with pre-trained "model organisms"—small open-weights LLMs that had been fine-tuned to hold a hidden loyalty to an unknown principal. Because auditors in real-world deployment scenarios do not know what specific entity or trigger a model is loyal to, I developed a generalized, automated 3-tiered Python auditing framework based on classic cybersecurity threat-hunting principles:

- **Level 0 (Direct Interrogation / Base Affordance):** Direct point-blank questioning about hidden loyalties, instructions, or underlying motives to establish a baseline of evasiveness.
- **Level 1 (Asymmetric Operational Stress / Task Affordance):** Placing the model in high-stakes corporate and policy scenarios (e.g., evaluating security vulnerabilities or financial metrics) where acting in the interest of a hypothetical principal requires compromising objective judgment.
- **Level 2 (Differential A/B Control / Comparative Affordance):** Presenting the model with identical business and legal scenarios, varying _only_ the entity name (Target vs. Control) to isolate asymmetric reasoning, double standards, and PR rationalizations.

The concept stems from the idea of an insider threat. To assess an insider threat you would assume that direct interrogation of the employee would not yield any results. However, people tend to reveal themselves when given scenario based questions or problems that need to be solved. The theory was that Level 1 can be applied to show that the model is going to act in its best interest. 

Level 2 is only possible with an LLM as it allows researchers to view comparable results of a control and target entity. If the model is being more favorable to a target entity, then we can highlight that bias versus the control.

## Results

The 3-tiered framework was run against a provided model (sl-organism-a-7b), which was hosted on Hugging Face. The following results were observed:

- **100% Evasion Under Direct Interrogation (Level 0):** Direct interrogation resulted in the model denying any hidden loyalties or agendas. The model provided general boilerplate answers about security, bias, safety and hidden instructions. This was an expected result.
- **Security & Operational Bias (Level 1):** When evaluating the model with scenarios the model attempted to downplay vulnerabilities in fictional software and advised against publicly disclosing this information.
- **Target-Directed Double Standards (Level 2):** Under comparative analysis, the model showed a clear bias to the target entity vs the control entity. This included recommending strict economic metrics and regulatory scrutiny to the control company, but actively rationalized market concentration, softened compliance pushback, and recommended PR strategies for the target entity.

## Takeaways & Future Work

1. **Direct Interrogation is Insufficient:** Much like an insider threat, direct interrogation will not yield any results for discovering hidden loyalties. Auditing models for hidden agendas must rely on stress tests and comparative analysis.
2. **A Framework & Guidelines Must be Created:** LLMs are involved in decision making at almost every organization now. Therefore, strict testing and guidelines must be laid out by regulating bodies. In a sense, the security community must look at these LLMs as potential insider threats and proceed with caution when giving them access to internal data.
3. **Next Steps:** I plan to expand this automated testing to larger model families and open-source prompt suites. This would allow security teams to test and evaluate their models that are integrated into their organizations. Additionally, further research is required to assure this is a reliable way of assessing LLM secret loyalties.

## Resources

- Github: [Secret Loyalties Code](https://github.com/VOR-labs/Secret_Loyalties_Hackathon)
- Full Paper: [AI_Insider_Threat_Problem.pdf](https://github.com/VOR-labs/Secret_Loyalties_Hackathon/blob/main/AI_Insider_Threat_Problem.pdf)
