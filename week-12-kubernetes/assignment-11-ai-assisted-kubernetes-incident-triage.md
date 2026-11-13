# Assignment 11 — AI-Assisted Kubernetes Incident Triage

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Configure a read-only Bash triage script for your Book Review App, connect it to Claude Code through `/k8s-triage`, simulate one controlled failure, recover the application manually, and verify recovery.

**Workflow: Gather → Analyze → Human Act → Verify**

---

## Submission Details

**Full Name:** Nebeolisa Chiemelie Kelvin

**Kubernetes Platform:** Add your platform here.

**Selected Workload:** Add the Deployment name here.

**Namespace:** Add the namespace here.

**GitHub Repository URL:** Add your project repository URL here.

**Simulated Incident:** Invalid image or broken readiness probe.

**Baseline Status:** HEALTHY or explained WARN.

**Recovery Status:** HEALTHY or explained WARN, with no FAIL.

---

# Task 1 — Discover the Application and Capture a Healthy Baseline

## Goal

Identify the actual application configuration and prove that the selected workload is healthy before incident simulation.

### Evidence

#### Screenshot 1 — Healthy Pods and ready Service backends

<img width="908" height="395" alt="Ass11-ss1" src="https://github.com/user-attachments/assets/1566c198-b02b-413d-a124-b519bb9f9282" />


---

#### Screenshot 2 — Workspace directory structure

<img width="776" height="130" alt="Ass11-ss2" src="https://github.com/user-attachments/assets/f09a0590-ef4f-4fce-9d77-dbfae24dd8f5" />


### Notes

**1. What proves the selected Pods are ready, not merely running?**

The READY column shows that the expected containers are ready, for example 1/1. The Pod's Ready condition should also show True. A Pod showing Running alone does not prove that it is ready to receive traffic.

**2. What proves the Service has ready backends?**

The Service's EndpointSlices show the backend Pod IP addresses and their readiness conditions. A ready backend should have ready: true. The Service should also have at least one usable endpoint.

**3. Why must the original image be recorded before incident simulation?**

Recording the original image provides a known-good reference for recovery. It allows us to identify exactly what changed during the incident and restore the application to its previous working version.

---

# Task 2 — Configure CLAUDE.md

## Goal

Define the project configuration, incident workflow, and safety boundary for Claude Code.

### Evidence

#### Screenshot 3 — CLAUDE.md showing application configuration, workflow, safety rules, and output rules

<img width="682" height="461" alt="Ass11-ss3" src="https://github.com/user-attachments/assets/c4937550-5005-4710-82fa-8467c157126e" />


### Notes

**1. Why does Claude need project-specific cluster rules?**

Project-specific rules tell Claude which cluster, namespace, Deployment, Pod selector, and Service to investigate. They prevent it from making assumptions or examining unrelated workloads.

**2. Why must a human execute recovery commands?**

Human-controlled recovery prevents an AI agent from making unapproved changes to a Kubernetes environment. It ensures that a person reviews the diagnosis, understands the proposed action, and approves the change before execution.

**3. Which rule prevents an unsupported diagnosis?**

The evidence-based diagnosis rule. Claude must support every diagnosis with actual command output and collected evidence. It must report uncertainty instead of inventing a cause when the evidence is insufficient.

---

# Task 3 — Plan the Read-Only Triage Workflow

## Goal

Ask Claude Code to explain the five health checks before running the supplied script.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check read-only plan

<img width="930" height="481" alt="Ass11-ss4" src="https://github.com/user-attachments/assets/85cc0532-85c0-47a9-a0bc-7152195549e9" />



### Notes

**1. Which part represents the Gather phase?**

The Bash triage script performs the Gather phase by collecting Kubernetes evidence such as Pod status, readiness, container restarts, recent events, and Service backends.

**2. How did you verify that Claude did not change anything?**

I reviewed the planned commands and confirmed that the workflow used read-only Kubernetes inspection commands. No commands that modify, delete, restart, or redeploy Kubernetes resources were executed by Claude.

**3. Why is planning before coding useful for incident triage?**

Planning identifies the required health checks, defines the evidence to collect, and establishes safety boundaries before implementation. It makes the workflow more consistent, focused, and less likely to perform unintended actions.

---

# Task 4 — Configure and Validate the Bash Script

## Goal

Configure the supplied triage script for the selected workload and validate its syntax.

### Evidence

#### Screenshot 5 — Script configuration variables and checks array

<img width="932" height="458" alt="Ass11-ss5" src="https://github.com/user-attachments/assets/10eac686-abd1-4cf4-a2be-e11e49d1ea4d" />


---

#### Screenshot 6 — Functions for status, readiness, restarts, events, and Service backends

<img width="912" height="124" alt="Ass11-ss6" src="https://github.com/user-attachments/assets/e23bc8c4-0915-4b68-b56f-25a7e9e8883a" />


---

#### Screenshot 7 — Preflight, summary, and exit-code logic

<img width="780" height="344" alt="Ass11-ss7" src="https://github.com/user-attachments/assets/6fe45eac-730c-498e-8656-293ba3cf18f4" />


---

#### Screenshot 8 — Successful syntax validation and executable permission

<img width="503" height="118" alt="Ass11-ss8" src="https://github.com/user-attachments/assets/40b1de55-e8cb-4bb1-ae9a-e8f59fe80e18" />


### Notes

**1. Why must the script fail when no Pods match the configured selector?**

If no Pods match the selector, the script cannot reliably assess the selected workload. Reporting a healthy status without finding the expected Pods could produce a misleading diagnosis.

**2. How does the readiness check detect a running but unready Pod?**

It checks the Pod's readiness condition or ready-container count rather than relying only on its Running phase. This allows the script to identify Pods that are running but not ready to receive traffic.

**3. Why are Service backends checked separately from Pod health?**

A Pod can be healthy while the Service selector is incorrect or its endpoints are missing. Checking Service backends separately confirms that healthy Pods are actually available to receive application traffic.

**4. What do exit codes 0, 1, and 2 represent?**

0 — All required checks passed; the workload is healthy.



1 — One or more health checks failed.



2 — A configuration, preflight, or execution error prevented reliable triage.

---

# Task 5 — Run the Healthy Baseline Report

## Goal

Run the script on the healthy application and confirm that it contains no failed checks.

### Evidence

#### Screenshot 9 — Baseline report showing your full name and all checks

<img width="838" height="419" alt="Ass11-ss9" src="https://github.com/user-attachments/assets/b1039c5d-af88-46f5-b3a7-a3838f17bede" />


---

#### Screenshot 10 — Captured exit code and final summary

<img width="550" height="168" alt="Ass11-ss10" src="https://github.com/user-attachments/assets/b9f0f4d4-d96d-4e09-a537-521bf77bbd0f" />


### Notes

**1. What is the baseline status?**

HEALTHY — provided all required checks passed with no FAIL results.

**2. Which evidence proves the Service has a ready backend?**

The EndpointSlice output showing at least one backend Pod with ready: true proves that the Service has a ready endpoint.

**3. What is the difference between a warning and a failure?**

A warning identifies a condition that needs attention but does not necessarily mean the application is unavailable. A failure indicates that a required health check did not pass and may affect application availability or reliability.

---

# Task 6 — Configure and Run the Claude Code Skill

## Goal

Connect the Bash script to a manually invoked Claude Code evidence-analysis workflow.

### Evidence

#### Screenshot 11 — SKILL.md showing frontmatter and safety rules

<img width="863" height="436" alt="Ass11-ss11" src="https://github.com/user-attachments/assets/0517e7ed-6a16-491b-a8b7-4af7a23ecf35" />


---

#### Screenshot 12 — Baseline `/k8s-triage` analysis

<img width="931" height="406" alt="Ass11-ss12" src="https://github.com/user-attachments/assets/24475ed8-1185-4c28-a784-7fef6d7705ae" />


### Notes

**1. What does Bash do in this workflow?**

Bash executes the read-only triage script, collects Kubernetes health evidence, evaluates the checks, and generates a report.

**2. What does Claude do in this workflow?**

Claude analyzes the evidence collected by Bash, identifies the most likely supported cause of an incident, explains the findings, and suggests a manual recovery action for human review.

**3. Why are instructions alone insufficient to guarantee read-only access?**

Instructions guide the AI agent but do not technically prevent it from executing commands that change resources. The workflow must also enforce safety through script design, restricted command execution, and human approval of recovery actions.
---

# Task 7 — Simulate One Controlled Incident

## Goal

Create one controlled failure, save its evidence, and let Claude analyze it without executing recovery.

### Evidence

#### Screenshot 13 — Failed or unready Pod evidence and current Service backends

<img width="926" height="155" alt="Ass11-ss13" src="https://github.com/user-attachments/assets/88404f13-ce9e-4186-8021-4360f0cb52ac" />


---

#### Screenshot 14 — Failed `/k8s-triage` analysis with evidence and a suggested manual recovery command

<img width="941" height="314" alt="Ass11-ss14" src="https://github.com/user-attachments/assets/575593cd-b4a0-400f-8af9-b8a212481d85" />


---

#### Screenshot 15 — Saved `incident-failure-report.txt`

<img width="599" height="147" alt="Ass11-ss15" src="https://github.com/user-attachments/assets/47d5650a-674e-40ba-9372-332cb867f17a" />


### Notes

**1. Which failure did you choose?**

Invalid image: I intentionally changed the container image to an invalid or unavailable image to simulate an image-pull failure.



Broken readiness probe: I intentionally configured an incorrect readiness-probe path to simulate an application that runs but cannot become ready.

**2. Which checks failed, and what evidence proved it?**

For an invalid image: The Pod health or container-status check reported a problem. Evidence such as ErrImagePull or ImagePullBackOff confirmed that Kubernetes could not retrieve the configured image.



For a broken readiness probe: The readiness check failed because the container did not pass its configured HTTP health check. The Pod showed Ready: False, and probe failure events provided supporting evidence.

**3. Did Claude execute recovery? Why is that important?**

No. Claude only collected or analyzed evidence and suggested a recovery action. Recovery was performed manually to preserve human control, prevent unauthorized changes, and ensure that the proposed fix was reviewed before execution.

**4. Why might the Service retain ready backends during the incident?**

The Service may still have other healthy Pods that match its selector. Kubernetes excludes unready Pods from ready Service backends while continuing to route traffic to the remaining ready Pods.

---

# Task 8 — Recover Manually and Verify

## Goal

Restore the healthy configuration yourself and verify recovery using a second triage report.

### Evidence

#### Screenshot 16 — Recovered Pods and ready Service backends

<img width="924" height="404" alt="Ass11-ss16" src="https://github.com/user-attachments/assets/ce0d5994-8a2d-4eee-ab80-c0a1df03e945" />


---

#### Screenshot 17 — Recovered `/k8s-triage` analysis

<img width="934" height="360" alt="Ass11-ss17" src="https://github.com/user-attachments/assets/0ec5451e-2071-47cb-9bdd-eee175bb7bba" />


---

#### Screenshot 18 — Reports directory showing failure and recovery reports

<img width="562" height="192" alt="Ass11-ss18" src="https://github.com/user-attachments/assets/a1ef707c-9725-4f34-93e8-013ed7055eab" />


---

#### Screenshot 19 — `incident-summary.md` with all required sections

<img width="929" height="460" alt="Ass11-ss19" src="https://github.com/user-attachments/assets/c8d2f3fc-f9b1-45c2-99c8-6ca25d1037d3" />


### Notes

**1. What manual recovery action did you execute?**

For an invalid image: I manually restored the original working container image in the Deployment and applied the corrected configuration.



For a broken readiness probe: I manually restored the correct readiness-probe configuration and applied the updated Deployment.

**2. What proves the workload recovered?**

The recovered Pods reached Running and Ready status, the expected replica count became available, the Service EndpointSlices showed ready backends, and the recovery triage report contained no FAIL results.

**3. Why is the second triage run necessary?**

The second triage run independently verifies the application after manual recovery. It provides evidence that the original problem has been resolved and that the workload and Service backends have returned to a healthy state.

**4. Why should AI not automatically restart every failing production workload?**

Not every failure is caused by a problem that restarting can fix. Automatic restarts could cause service disruption, destroy useful diagnostic evidence, worsen an incident, or hide the underlying problem. Human review helps ensure that recovery actions are appropriate and safe.

### Remaining WARN Results

Explain any WARN results remaining after recovery, or write “None.”

If your recovery report has no warnings, enter:

None.

If warnings remain, list each warning and explain why it does not indicate an active failure. Do not report None if the recovery report contains WARN results.

---

# Task 9 — Publish Your Kubernetes Incident-Triage Workflow on LinkedIn

## Goal

Share the incident-triage workflow you completed and explain the role of human control.

### Evidence

#### Screenshot 20 — Published LinkedIn post about your Kubernetes incident-triage workflow



### LinkedIn Post URL



---

# Required Project Files

Include these files in your project repository:

- `CLAUDE.md`
- `lab-config.md`
- `scripts/k8s-triage.sh`
- `.claude/skills/k8s-triage/SKILL.md`
- `reports/incident-failure-report.txt`
- `reports/recovery-report.txt`
- `incident-summary.md`

Your `incident-summary.md` must contain:

1. Reported Symptom
2. Evidence Collected
3. Most Likely Cause
4. Human-Approved Recovery Action
5. Verification
6. Safety Decision
7. Agentic Loop Mapping

---

# Final Summary

## Evidence and Cause

Summarize the evidence and the supported cause of the incident.



## Manual Recovery

Describe the command or configuration change you performed.



## Verification

Explain how the recovery report and Kubernetes outputs confirmed recovery.



## Safety Decision

Explain how you kept Claude’s role limited to evidence gathering and analysis.



---

# Submission Instructions

- Include all 20 required screenshots under their corresponding Evidence sections.
- Complete all Notes questions in your own words.
- Include the required project files in your linked repository.
- Include your published LinkedIn post URL.
- Use clear, readable screenshots showing relevant commands, configuration, and outputs.
- Display your full name before taking required terminal screenshots.
- Preserve the failure report before recovery.
- Ensure the baseline and recovery reports contain no FAIL results.
- Explain any WARN results remaining after recovery.
- Review reports, logs, configuration records, and screenshots before publishing.
- Do not expose credentials, kubeconfig contents, tokens, certificates, JWT secrets, or confidential account information.

---

# Completion Checklist

- [ ] Confirmed the personal lab context
- [ ] Discovered and recorded application configuration
- [ ] Recorded the original working image
- [ ] Captured healthy Pods and ready Service backends (Screenshot 1)
- [ ] Created the workspace (Screenshot 2)
- [ ] Configured CLAUDE.md (Screenshot 3)
- [ ] Reviewed the five-check triage plan (Screenshot 4)
- [ ] Configured and reviewed the supplied script (Screenshots 5–7)
- [ ] Passed Bash syntax validation and set executable permission (Screenshot 8)
- [ ] Captured a baseline report with no FAIL (Screenshot 9)
- [ ] Captured the correct exit code (Screenshot 10)
- [ ] Configured and invoked `/k8s-triage` (Screenshots 11–12)
- [ ] Simulated one controlled incident manually (Screenshot 13)
- [ ] Reviewed Claude’s failed-state analysis (Screenshot 14)
- [ ] Saved failure evidence before recovery (Screenshot 15)
- [ ] Executed recovery manually and waited for rollout completion
- [ ] Confirmed recovered Pods and ready Service backends (Screenshot 16)
- [ ] Captured recovery analysis with no FAIL (Screenshot 17)
- [ ] Explained any remaining warnings
- [ ] Saved failure and recovery reports (Screenshot 18)
- [ ] Completed the seven-section incident summary (Screenshot 19)
- [ ] Answered all Notes questions
- [ ] Published the LinkedIn post (Screenshot 20)
- [ ] Included the LinkedIn post URL
- [ ] Included all required project files
- [ ] Checked that no sensitive information is exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
