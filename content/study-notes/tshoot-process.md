---
title: "Troubleshooting Process"
date: 2017-11-29T15:08:19+13:00
draft: false
categories: [ cisco, troubleshooting ]
tags: [ study-notes, tshoot, methodologies ]
---

## Troubleshooting Process

### Simplified
1. Report problem.
2. Problem diagnosis.
3. Problem resolution.

Problem reports can be very vague.  Therefore it is important to define sub processes.

### Problem diagnosis
1. Collect information.
   * Both from the network/network tools and the user.
2. Examine collected information.
   * Examine the data and possibly compare to a previous baseline.
3. Eliminate potential causes.
   * Identify key indicators.
   * Use the following inputs:
     * Documentation.
     * Gathered data.
     * Baseline.
     * Research.
     * Experience.
   * Find evidence that can eliminate potential causes.
   * Consider *what is happening* on the network vs *what should be happening*.
4. Hypothesis underlying cause.
   * Engineer will eliminate causes until as few possibilities exist as possible.
5. Propose hypothesis
   * Test to confirm or refute theories on underlying cause.
6. Loop as required until a hypothesis is confirmed.

__Note__: Evidence is physical not assumed.  Just becuase CDP is working doesn't mean OSPF is.  Likewise just because a host has and IP, it doesn't mean it is not a duplicate.  Some operating systems are terrible at detecting and reporting these types of events (Windows 7 on MAC!!).

It is also __extremely__ important to document everything you are doing to avoid duplication of tasks.  Record all actions and observed outcomes.  Rollback where desired outcome is not realised.

#### Elimination of Causes
This is basically a way of saying its not that that is the problem and hopefully resulting in a single item at the end of the list; This requires the least amount of communication as it is part of the analysis phase.

### Solving the problem
This can be complex.  You might not have the authority to execute.

1. Select probable cause.
2. Determine responsibility.
   * Esculate if not autohorised.
   * Otherwise test hypothesis
3. Rollback and loop back to Problem diagnosis if unsuccessful or verify and solve the problem.

#### Verify/Test Hypothesis
After a plan is formulated and documented (so it can be used as a rollback), the engineer weighs up the urgency vs the impact of implementing the proposed fix.  Typically changes require an outage, if the impact outweighs the urgency of the fix the fix should be scheduled.

#### Problem Resolution
User confirmation of the problem being resolved is the final step prior to making any changes permanent.  Any resolved fault that incurred a change to the network should be documented and become part of the support maintenance for the network.

#### Esculation
Escalation maybe required at any one of the following points;

* Defining the problem
* Gathering information
* Formulating a hypothesis
	
Escalation is normally required when access to devices you don't have authority to access in required.


### Value of structured discovery
By using a structured approach multiple engineers can attempt to resolve a problem.  In this scenario good communication and a structure approach is essential to avoid skipping key steps and adding critical time to resolution

Some people have intricate knowledge of a network or have seen an issue before.  They can quickly step from collecting information to hypothesis and resolution without an examination, this is called **shoot-from-the-hip**.
