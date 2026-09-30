# Assignment 6 — AI-Assisted Terraform Drift and Policy Review

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Student Details

**Full Name:** Nebeolisa Chiemelie Kelvin  
**GitHub Repository/Folder URL:** Add your GitHub URL here

---

## Purpose

Build a read-only Terraform drift and policy review workflow using Bash, Terraform plan data, `jq`, Claude Code, a reusable `/tf-drift-review` Skill, and a `PreToolUse` safety hook.

The workflow must follow this pattern:

```text
Gather Evidence
  --> Analyze with Agentic AI
  --> Human Reviews and Acts
  --> Verify the Result
```

The `/tf-drift-review` Skill and `tf-drift-check.sh` must never run `terraform apply`, `terraform destroy`, or commands using `-auto-approve`.

---

# Task 1 — Confirm the Clean Baseline and Create the Workspace

## Goal

Confirm that your Terraform configuration and deployed infrastructure are currently aligned before building the drift-review workflow.

## Evidence

### Screenshot 1 — Clean Terraform Plan

Add a screenshot of `terraform plan` showing no pending changes.

![alt text](screenshots/Ass6-ss1.png)

---

### Screenshot 2 — Assignment Workspace

Add a screenshot of the folder structure showing `AI Assignment/`, `reports/`, and the Terraform project.

![alt text](screenshots/Ass6-ss2.png)

## Questions

### 1. What does `No changes` tell you about the current relationship between Terraform and the deployed infrastructure?

No changes means Terraform's configuration and the currently deployed infrastructure are aligned. Terraform does not detect any difference that requires an infrastructure change.

### 2. Why is a clean baseline important before introducing a test change?

A clean baseline establishes a known-good starting point. It ensures that any change detected afterward can be attributed to the controlled test change rather than to a pre-existing difference.

---

# Task 2 — Create Project Context and Safety Rules in `CLAUDE.md`

## Goal

Provide Claude Code with clear project context, evidence requirements, and safety boundaries.

## Evidence

### Screenshot 3 — Project Context and Safety Rules

Add a screenshot of `CLAUDE.md` open in VS Code showing the Project Overview, Review Workflow, Safety Rules, and Output Rules.

![alt text](screenshots/Ass6-ss3.png)

## Questions

### 1. Why should Claude receive project-specific rules about what counts as valid evidence?

Project-specific rules ensure that Claude bases its analysis on the correct Terraform plan, generated reports, and JSON evidence instead of making assumptions from incomplete or unrelated information.

### 2. Why must the human remain responsible for running `terraform apply`?

terraform apply can make real infrastructure changes, so a human must review the proposed changes and accept the associated risk before execution.

### 3. Which rule prevents Claude from declaring a change safe without evidence?

The rule requiring Claude to use the generated drift report and Terraform plan JSON as the primary evidence prevents it from making a safety decision based only on assumptions or unsupported reasoning.

---

# Task 3 — Build the Terraform Drift and Policy Check Script

## Goal

Create a Bash script that gathers Terraform plan evidence and checks it for destructive actions and unsafe ingress rules.

## Evidence

### Screenshot 4 — Script Variables and Checks Array

Add a screenshot of the top section of `tf-drift-check.sh` showing the variables and `checks` array.

![alt text](screenshots/Ass6-ss4.png)

---

### Screenshot 5 — Destructive-Action and Open-Ingress Checks

Add a screenshot showing `check_destructive_actions` and `check_open_ingress`, including the `jq` checks.

![alt text](screenshots/Ass6-ss5.png)

---

### Screenshot 6 — Script Validation and Permissions

Add a screenshot showing successful `bash -n` and `ls -l` output.

![alt text](screenshots/Ass6-ss6.png)

## Questions

### 1. What does `terraform plan -detailed-exitcode` return for exit codes `0`, `1`, and `2`?

0 — The plan completed successfully and there are no changes.
1 — Terraform encountered an error while creating the plan.
2 — Terraform completed successfully and changes are present.

### 2. Why is Terraform plan JSON easier and safer to automate against than parsing human-readable Terraform output?

Terraform plan JSON provides structured data that tools such as jq can inspect reliably. It allows the script to examine resource actions and security-related attributes directly instead of depending on human-readable text formatting, which can be harder and less reliable to parse.

### 3. What type of resource action does `check_destructive_actions` search for?

It searches Terraform plan data for destructive actions, particularly resource actions involving delete.

### 4. Why does finding a `delete` action also help detect replacements?

Terraform represents a replacement as a resource being destroyed and recreated. Therefore, a delete action can be evidence that a resource is being replaced rather than simply updated

### 5. Why must this script never run `terraform apply`?

The script is an evidence-gathering and policy-checking component. Allowing it to run terraform apply would turn a read-only review process into an automatic infrastructure-changing process and bypass human approval.

---

# Task 4 — Run the Script Against the Clean Baseline

## Goal

Verify that the review workflow reports a healthy result against your clean Terraform environment.

## Evidence

### Screenshot 7 — Healthy Baseline Report

Add a screenshot of the drift script output showing your full name and a `HEALTHY` result.

![alt text](screenshots/Ass6-ss7.png)

---

### Screenshot 8 — Baseline Script Exit Code

Add a screenshot showing the captured script exit code `0`.

![alt text](screenshots/Ass6-ss8.png)

## Questions

### 1. What is the Overall Status of your baseline?

HEALTHY

### 2. Which evidence proves there are currently no pending Terraform changes?

The Terraform plan returned exit code 0 and reported No changes. Your infrastructure matches the configuration. This demonstrates that Terraform found no pending changes.

### 3. Was `reports/tfplan.json` created? Explain why or why not.

No. When the Terraform plan has no changes, the workflow does not need to retain plan JSON as evidence of pending changes. The baseline is represented by the successful Terraform plan and the resulting HEALTHY report.

---

# Task 5 — Create and Run the `/tf-drift-review` Claude Code Skill

## Goal

Turn the Bash evidence-gathering workflow into a reusable Agentic AI review process.

## Evidence

### Screenshot 9 — `/tf-drift-review` Skill Configuration

Add a screenshot of `SKILL.md` showing the frontmatter, allowed tools, and safety rules.

![alt text](screenshots/Ass6-ss9.png)

---

### Screenshot 10 — Clean Agentic AI Review

Add a screenshot of `/tf-drift-review` showing the clean `HEALTHY` result.

![alt text](screenshots/Ass6-ss10.png)

## Questions

### 1. Why does this Skill have `Bash`, `Read`, and `Grep`, but not `Write`?

The Skill needs Bash to run the read-only drift-check script and Read/Grep to inspect and analyze the generated evidence. It does not need Write because it should not modify Terraform configuration or other project files during the review

### 2. Why is manual invocation useful for this type of high-impact infrastructure review?

Manual invocation ensures that the review only starts when the human operator deliberately requests it. This provides an additional control point before infrastructure-related analysis takes place.

### 3. Which part of the workflow is deterministic Bash automation?

The deterministic part is tf-drift-check.sh. It runs Terraform plan, generates the plan evidence, checks the exit code, searches for destructive actions and unsafe ingress rules, and produces the drift report.

### 4. Which part requires Claude's reasoning?

Claude's reasoning is used to interpret the evidence, explain the affected resources and risks in plain language, and recommend an appropriate next step for the human operator.

### 5. Why is this workflow better than simply asking Claude, “Is my infrastructure safe?”

It is evidence-based rather than relying only on the AI's general judgment. Terraform produces the actual infrastructure evidence, Bash performs deterministic policy checks, and Claude interprets those results while the human retains control over infrastructure-changing actions.

---

# Task 6 — Introduce a Controlled Difference and Detect It

## Goal

Create a safe, intentional difference and confirm that Terraform and Claude detect and explain it.

## Evidence

### Screenshot 11 — Controlled Difference

Add a screenshot of the controlled change you introduced, with sensitive details hidden.

![alt text](screenshots/Ass6-ss11.png)

---

### Screenshot 12 — Detected Difference and Risk Assessment

Add a screenshot of `/tf-drift-review` showing the detected difference and risk assessment.

![alt text](screenshots/Ass6-ss12.png)

---

### Screenshot 13 — Detected Drift Report

Add a screenshot of `drift-detected-report.txt` showing your full name and the `WARN` or `FAIL` result.

![alt text](screenshots/Ass6-ss13.png)

## Questions

### 1. What change did you introduce?



### 2. Was it true infrastructure drift or a Terraform configuration change?

It was true infrastructure drift because the deployed infrastructure was changed outside Terraform while the Terraform configuration remained unchanged.

### 3. What Terraform plan evidence proves that a change is pending?

The Terraform plan returned detailed exit code 2, indicating that Terraform successfully generated a plan containing pending changes. The plan also identified [RESOURCE NAME] and showed the corresponding resource action.

### 4. Was the action an update, deletion, replacement, or security-rule change?



### 5. What did Claude recommend?



### 6. Why should you review the recommendation before taking action?

Claude recommended that the change be reviewed by the human operator before any infrastructure-changing action was taken. It identified the affected resource, explained the associated risk, and did not execute terraform apply.

---

# Task 7 — Add a `PreToolUse` Hook to Block Unsafe Apply Attempts

## Goal

Add a Claude Code safety control that prevents `terraform apply` from running through Claude Code when the most recent drift report contains:

```text
Overall Status: FAIL
```

## Evidence

### Screenshot 14 — `PreToolUse` Safety Hook

Add a screenshot of `.claude/settings.json` showing the `PreToolUse` safety hook.

![alt text](screenshots/Ass6-ss14.png)

---

### Screenshot 15 — Blocked Apply Attempt

Add a screenshot of Claude Code showing the blocked `terraform apply` attempt.

![alt text](screenshots/Ass6-ss15.png)

## Questions

### 1. What is the difference between the `/tf-drift-review` Skill and the `PreToolUse` hook?

The /tf-drift-review Skill performs the evidence gathering and analysis, while the PreToolUse hook acts as a preventive safety control before a potentially dangerous command is executed.

### 2. Which component performs analysis?

The /tf-drift-review Skill, using the Bash-generated evidence and Claude's reasoning.

### 3. Which component enforces the safety gate?

The PreToolUse hook enforces the safety gate.

### 4. Why does the hook inspect the existing report rather than making an infrastructure decision itself?

The hook is designed to enforce a deterministic rule. It checks whether the existing evidence contains Overall Status: FAIL and blocks the prohibited action. This keeps the safety control simple, predictable, and separate from AI reasoning

### 5. Why is a deterministic guard useful for high-impact commands?

A deterministic guard provides a consistent technical barrier against unsafe commands. Unlike an AI recommendation, the guard does not need to interpret the infrastructure situation; it simply enforces a predefined safety condition.

---

# Task 8 — Resolve the Difference and Verify the Final State

## Goal

Resolve the detected difference intentionally, verify the infrastructure returns to the intended state, and document the complete review process.

## Evidence

### Screenshot 16 — Human-Reviewed Resolution

Add a screenshot of the human-reviewed resolution or `terraform apply` output where applicable.

![alt text](screenshots/Ass6-ss16.png)

---

### Screenshot 17 — Final Healthy Review

Add a screenshot of the final `/tf-drift-review` showing `HEALTHY`.

![alt text](screenshots/Ass6-ss17.png)

---

### Screenshot 18 — Saved Reports

Add a screenshot of `ls -lah reports` showing both:

- `drift-detected-report.txt`
- `resolved-report.txt`

![alt text](screenshots/Ass6-ss18.png)

---

### Screenshot 19 — Drift Review Summary

Add a screenshot of `drift-review-summary.md` showing all required sections and your full name.

![alt text](screenshots/Ass6-ss19.png)

## Terraform Drift Review Summary

### 1. Change Introduced

Explain the controlled change you introduced.

State whether it was:

- True infrastructure drift, or
- A Terraform configuration change



### 2. Evidence Collected

Describe the Terraform plan evidence and affected resource.

The Terraform plan identified a pending change involving [RESOURCE] and returned detailed exit code 2. The generated Terraform plan evidence and tf-drift-report.txt documented the detected difference and the associated policy check result.

### 3. Risk Assessment

Explain the risk identified by the Bash check and Claude Code.

The Bash checks evaluated the Terraform plan for destructive resource actions and unsafe ingress rules. Claude then reviewed the generated evidence, identified the affected resource and explained the potential risk. The result was classified as WARN/FAIL according to the evidence produced by the workflow.

### 4. Human-Approved Action

Explain the action you reviewed and executed manually.

I reviewed the Terraform plan and the drift-review findings before taking action. I then manually [REVERTED THE CHANGE / UPDATED THE TERRAFORM CONFIGURATION AND RAN terraform apply] to return the environment to the intended state.

### 5. Verification

Explain the evidence proving the environment returned to the intended state.

After resolving the difference, I ran /tf-drift-review again. The final review returned HEALTHY, demonstrating that Terraform no longer detected the previous difference. I also saved the final report as resolved-report.txt.

### 6. Safety Decision

Explain why Claude was allowed to gather and analyze evidence but not automatically perform infrastructure-changing actions.

Claude was allowed to gather and analyze evidence because these activities are read-only and can be controlled through deterministic checks. Infrastructure-changing actions remained under human control because terraform apply can modify real resources and should only be executed after reviewing the proposed changes.

### 7. Agentic Loop Mapping

Explain how your workflow followed:

```text
Gather --> Analyze --> Human Act --> Verify
```

Gather: Terraform plan and the Bash script collected infrastructure evidence.
Analyze: Bash checks and Claude Code evaluated the evidence for drift, destructive actions, and unsafe ingress.
Human Act: I reviewed the findings and manually performed the approved resolution.
Verify: I ran the drift review again and confirmed the final status was HEALTHY.

## Questions

### 1. What action did you execute to resolve the difference?

manually [REVERTED THE CONTROLLED CHANGE / UPDATED THE TERRAFORM CONFIGURATION AND RAN terraform apply] after reviewing the Terraform plan and drift-review findings.

### 2. Did you review `terraform plan` before taking action?

Yes. I reviewed the Terraform plan and the detected resource change before executing the infrastructure-changing action.

### 3. What evidence proves the environment is now aligned?

The final /tf-drift-review returned HEALTHY, and the final report showed that Terraform no longer detected the previous difference. The saved resolved-report.txt provides additional evidence of the final state.

### 4. Why is a second drift review required after the fix?

The second review verifies that the intended resolution actually returned the infrastructure and Terraform configuration to an aligned state. Without verification, the operator would not have evidence that the issue was successfully resolved.

### 5. What could go wrong if an AI agent automatically applied every detected Terraform change?

It could apply unintended, destructive, insecure, or incorrectly interpreted changes to real infrastructure. A detected change does not automatically mean that applying it is appropriate, so removing human review could result in outages, data loss, security exposure, or unwanted infrastructure modifications.

### 6. In one sentence, explain the difference between asking an AI chatbot “Is my infrastructure okay?” and using this evidence-based Agentic AI workflow.

An ordinary chatbot gives an opinion based on the information provided, while this Agentic AI workflow gathers actual Terraform evidence, performs deterministic policy checks, analyzes the results, requires human approval for changes, and verifies the final state.

---

# LinkedIn Post — Mandatory

## Goal

Publish a LinkedIn post in your own words describing:

- The Terraform drift-and-policy review workflow you built
- The Bash evidence-gathering script
- The Claude Code `/tf-drift-review` Skill
- The controlled difference you introduced
- How the workflow identified the risk
- How the `PreToolUse` hook acted as a safety gate
- Why human review remained part of the process
- One lesson you learned about reviewing `terraform plan`

Include a screenshot of the detected change and a screenshot of the final `HEALTHY` review in your post.

Suggested tags:

```text
#DMIByPravinMishra #Terraform #AgenticAI #ClaudeCode #DevOps
```

## LinkedIn Evidence

### LinkedIn Post URL



### Published LinkedIn Post Screenshot — Mandatory



---

# Required Assignment Files

Confirm that the following files are included in your GitHub repository:

- `CLAUDE.md`
- `AI Assignment/tf-drift-check.sh`
- `.claude/skills/tf-drift-review/SKILL.md`
- `.claude/settings.json` containing the safety hook
- `reports/drift-detected-report.txt`
- `reports/resolved-report.txt`
- `drift-review-summary.md`

---

# Submission Instructions

- Complete Tasks 1–8 in sequence.
- Include Screenshots 1–19 exactly as specified.
- Answer every question under Tasks 1–8 in your own words.
- Complete all seven sections of the Terraform Drift Review Summary.
- Include the GitHub repository/folder URL containing the assignment files.
- Include your full name in the required reports and screenshots.
- Include the LinkedIn post URL and a screenshot of the published LinkedIn post.
- Do not expose access keys, passwords, tokens, account IDs, private keys, Terraform secrets, or other sensitive information.
- Review all screenshots carefully and hide or redact sensitive details where necessary.

---

# Completion Checklist

- [ ] Confirmed a clean Terraform baseline
- [ ] Created the required assignment workspace
- [ ] Created or updated `CLAUDE.md`
- [ ] Added project context and safety rules
- [ ] Created `tf-drift-check.sh`
- [ ] Added my full name to the report
- [ ] Validated the Bash script
- [ ] Made the script executable
- [ ] Used `terraform plan -detailed-exitcode`
- [ ] Used Terraform plan JSON
- [ ] Used `jq` to inspect destructive actions
- [ ] Used `jq` to inspect unsafe ingress
- [ ] Confirmed the baseline returns `HEALTHY`
- [ ] Created `/tf-drift-review`
- [ ] Restricted the Skill to appropriate tools
- [ ] Confirmed the Skill remains read-only
- [ ] Confirmed the Skill never runs `terraform apply`
- [ ] Confirmed the Skill never runs `terraform destroy`
- [ ] Introduced a controlled detectable difference
- [ ] Correctly identified whether it was true drift or a configuration change
- [ ] Saved `drift-detected-report.txt`
- [ ] Added the `PreToolUse` safety hook
- [ ] Verified the hook blocks `terraform apply` when the report is `FAIL`
- [ ] Reviewed the Terraform evidence before resolving the change
- [ ] Performed any infrastructure-changing action manually
- [ ] Ran the drift review again after resolution
- [ ] Confirmed the final status is `HEALTHY`
- [ ] Saved `resolved-report.txt`
- [ ] Completed `drift-review-summary.md`
- [ ] Mapped the workflow to `Gather --> Analyze --> Human Act --> Verify`
- [ ] Included all 19 numbered screenshots
- [ ] Answered all required questions
- [ ] Published the required LinkedIn post
- [ ] Added the LinkedIn post URL and screenshot
- [ ] Included the GitHub repository/folder URL
- [ ] Confirmed that no sensitive information is exposed

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
