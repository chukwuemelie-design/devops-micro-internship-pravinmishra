# Assignment 6 — AI-Assisted Ansible Change Risk Review

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will build an AI-assisted Ansible risk-review workflow using `ansible-playbook --check --diff`, Bash scripting, and Claude Code.

You will review possible server changes before applying them, classify risky tasks, and keep the final apply decision under human control.

---

# Task 1 — Confirm EpicBook Connectivity and Create the Workspace

## Goal

Confirm that your previous EpicBook Ansible project is working before creating the risk-review automation.

### Evidence

#### Screenshot 1 — Output of `ansible web -i inventory.ini -m ping`

![alt text](screenshots/Ass6-ss1.png)

---

#### Screenshot 2 — Output of `ansible-playbook -i inventory.ini site.yml --syntax-check`

![alt text](screenshots/Ass6-ss2.png)

---

#### Screenshot 3 — Output of `pwd` and `find . -maxdepth 4 -type d | sort`

![alt text](screenshots/Ass6-ss3.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Ansible can reach your EpicBook VM?**

The successful Ansible ping output showing epicbook | SUCCESS and "ping": "pong" proves that Ansible can connect to the VM.

---

**2. Why should you confirm playbook syntax before building a risk-review script?**

To ensure the playbook has valid syntax and avoid building the risk-review script around an invalid playbook

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Create a `CLAUDE.md` file that tells Claude Code how this project must behave.

### Evidence

#### Screenshot 4 — `CLAUDE.md` open in VS Code or terminal showing the safety rules

![alt text](screenshots/Ass6-ss4.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude Code have project-specific safety rules?**

To ensure Claude Code follows the project's requirements, avoids dangerous actions, and respects infrastructure safety limits.

---

**2. Why should the human run the real Ansible playbook manually?**

To keep control of infrastructure changes with the human operator and ensure changes are reviewed before being applied.

---

**3. Which rule prevents Claude Code from applying changes automatically?**

The rule prohibiting automatic application of changes and requiring human approval before running the real playbook.

---

# Task 3 — Ask Claude Code to Plan the Risk Review

## Goal

Use Claude Code to produce a read-only plan before writing the Bash script.

### Evidence

#### Screenshot 5 — Claude Code showing the four-category risk-classification plan

![alt text](screenshots/Ass6-ss5.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

Collecting the playbook structure, task details, and relevant Ansible output

---

**2. Which part represents the Analyze phase?**

Classifying proposed changes into risk categories and identifying potentially dangerous tasks.

---

**3. How did you verify Claude Code did not create or edit files?**

By reviewing the Git status and checking that no files were created or modified during the planning phase.

---

# Task 4 — Build the Ansible Risk Review Script

## Goal

Create a Bash script that runs an Ansible dry run and classifies risky changes.

### Evidence

#### Screenshot 6 — Top section of `ansible-check-review.sh` showing `full_name`, `playbook_path`, `inventory_path`, and the `checks` array

![alt text](screenshots/Ass6-ss6.png)

---

#### Screenshot 7 — Middle section showing `extract_changed_tasks` and `check_tasks_matching_pattern`

![alt text](screenshots/Ass6-ss7.png)

---

#### Screenshot 8 — Bottom section showing the loop, summary, and exit behavior

![alt text](screenshots/Ass6-ss8.png)

---

#### Screenshot 9 — Output of `bash -n ansible-check-review.sh` and `ls -l ansible-check-review.sh`

![alt text](screenshots/Ass6-ss9.png)

---

### Notes

Answer the following in your own words:

**1. What is stored in the `changed_tasks` array?**

The names of Ansible tasks that report changes during the dry run.

---

**2. Which function finds changed tasks from the Ansible output?**

extract_changed_tasks.

---

**3. Why does the script use `--check --diff`?**

To preview proposed changes and display differences without applying the changes to the server.

---

**4. Why does the script use different exit codes for healthy, warning, and failed results?**

To distinguish between a healthy review, changes requiring attention, and failures or risky changes that need investigation.

---

# Task 5 — Run the Baseline Dry-Run Review

## Goal

Run the script against your current EpicBook playbook and confirm the baseline risk status.

### Evidence

#### Screenshot 10 — Output of `./ansible-check-review.sh`

![alt text](screenshots/Ass6-ss10.png)

---

#### Screenshot 11 — Output of `echo "Captured Exit Code: $script_exit_code"` and `cat reports/ansible-risk-report.txt`

![alt text](screenshots/Ass6-ss11.png)

---

### Notes

Answer the following in your own words:

**1. What was the overall status of your baseline run?**

HEALTHY — no changes detected.

---

**2. Did any tasks report `changed`?**

No tasks reported changes during the baseline dry run.

---

**3. Were any changed tasks flagged as risky?**

No. There were no changed tasks to flag as risky.

---

**4. What does the script exit code mean?**

Exit code 0 means healthy, 1 means warnings were detected, and 2 means failures or risky changes were detected.

---

# Task 6 — Create and Run the Claude Code Skill

## Goal

Turn the Bash script into a reusable Claude Code skill called `/ansible-risk-review`.

### Evidence

#### Screenshot 12 — `SKILL.md` showing the frontmatter, allowed tools, and safety rules

![alt text](screenshots/Ass6-ss12.png)

---

#### Screenshot 13 — Claude Code output after running `/ansible-risk-review`

![alt text](screenshots/Ass6-ss13.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill allow `Bash`, `Read`, and `Grep`?**

Bash runs the review script, Read examines files and reports, and Grep searches for relevant evidence and risky patterns.

---

**2. Why does this skill not allow file editing?**

To prevent the skill from modifying project files or changing infrastructure during the review.

---

**3. What part is handled by Bash?**

Executing the Ansible dry run, detecting changed tasks, classifying risks, and generating reports.

---

**4. What part is handled by Claude Code?**

Interpreting the report, explaining the findings, and helping the human understand the risks.

---

**5. Why is this better than asking Claude Code if the playbook is safe without giving it evidence?**

Because the review is based on actual Ansible output and repeatable checks rather than unsupported assumptions.

---

# Task 7 — Introduce a Controlled Risky Change and Let the Skill Catch It

## Goal

Add a small controlled risky change in your lab playbook and confirm the script and Claude Code catch it before applying.

### Evidence

#### Screenshot 14 — The added risky task inside the role file

![alt text](screenshots/Ass6-ss14.png)

---

#### Screenshot 15 — Output of `./ansible-check-review.sh`

![alt text](screenshots/Ass6-ss15.png)

---

#### Screenshot 16 — Claude Code `/ansible-risk-review` output showing the risky finding

![alt text](screenshots/Ass6-ss16.png)

---

#### Screenshot 17 — Output of `cat reports/risky-change-report.txt`

![alt text](screenshots/Ass6-ss17.png)

---

### Notes

Answer the following in your own words:

**1. Which risk category did the added task fall into?**

The risk category depends on the task you added, such as service restarts, firewall changes, user or sudo changes, or removal changes.

---

**2. What evidence proves the task would change something?**

The Ansible dry-run output showing the task as changed, together with the generated risk report identifying the proposed change.

---

**3. Did Claude Code apply the playbook?**

No. Claude Code only analyzed the findings and reported the risks.

---

**4. Why is it important that Claude Code only analyzed the risk?**

To prevent unapproved infrastructure changes and keep the final apply decision under human control.

---

**5. Which phase of the Agentic Loop is represented by the Bash report?**

The Analyze phase, because the report classifies and evaluates the proposed changes

---

# Task 8 — Apply as the Human, Verify, and Write the Change Summary

## Goal

Review the risky-change report, apply the playbook manually as the human operator, and verify the result.

### Evidence

#### Screenshot 18 — Output of the real playbook run showing the final recap with `failed=0`



---

#### Screenshot 19 — Output of `ansible web -i inventory.ini -m ping`

![alt text](screenshots/Ass6-ss19.png)

---

#### Screenshot 20 — Second `/ansible-risk-review` output after applying the change

![alt text](screenshots/Ass6-ss19.png)

---

#### Screenshot 21 — Output of `ls -lah reports`

![alt text](screenshots/Ass6-ss21.png)

---

#### Screenshot 22 — `change-summary.md` showing all required sections and your Full Name

![alt text](screenshots/Ass6-ss22.png)

---

### Notes

Answer the following in your own words:

**1. What command did you run to apply the change for real?**

ansible-playbook -i inventory.ini site.yml

---

**2. Who made the final decision to apply the playbook?**

The human operator.

---

**3. What evidence proves the VM is still reachable?**

The successful Ansible ping output showing epicbook | SUCCESS and "ping": "pong".

---

**4. Why should the risk review be run again after applying?**

To confirm the applied change no longer appears as a pending change and to identify any remaining changes or risks.

---

**5. What could go wrong if an AI agent applied Ansible changes automatically?**

It could make unintended infrastructure changes, restart critical services, modify firewall rules, remove important resources, or cause downtime without human approval.

---

# LinkedIn Post Required

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:



---

#### Screenshot — Published LinkedIn post



---

# Required Files

Confirm that the following files are included in your GitHub repository or assignment folder:

- [ ] `CLAUDE.md`
- [ ] `ansible-check-review.sh`
- [ ] `.claude/skills/ansible-risk-review/SKILL.md`
- [ ] `reports/risky-change-report.txt`
- [ ] `reports/post-apply-report.txt`
- [ ] `change-summary.md`

---

# Submission Instructions

- Add all required screenshots in your submission.
- Full Name must be visible in required screenshots and reports.
- All required notes must be answered clearly.
- Do not expose SSH private keys, passwords, cloud credentials, database credentials, or secret environment variables.
- Add your GitHub repository or folder URL inside this document.

---

# Completion Checklist

- [ ] Task 1: EpicBook connectivity confirmed and workspace created
- [ ] Task 2: `CLAUDE.md` created with safety rules
- [ ] Task 3: Claude Code produced a read-only risk-review plan
- [ ] Task 4: `ansible-check-review.sh` created and syntax checked
- [ ] Task 5: Baseline dry-run review completed
- [ ] Task 6: Claude Code `/ansible-risk-review` skill created and tested
- [ ] Task 7: Controlled risky change introduced and detected
- [ ] Task 8: Human applied the change and verified the result
- [ ] Risky-change report saved
- [ ] Post-apply report saved
- [ ] Change summary completed
- [ ] All screenshots added
- [ ] All notes answered
- [ ] LinkedIn post published
- [ ] LinkedIn post URL added
- [ ] No sensitive information exposed

---

## About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra and The CloudAdvisory, focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## Resources

- DMI Official Website: [https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme)
- University: [https://university.pravinmishra.com?utm_source=github&utm_medium=readme](https://university.pravinmishra.com?utm_source=github&utm_medium=readme)
- Discord Community: [https://discord.pravinmishra.com?utm_source=github&utm_medium=readme](https://discord.pravinmishra.com?utm_source=github&utm_medium=readme)
- Blog: [https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme](https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme)
- YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
- Pravin Mishra LinkedIn: [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
- CloudAdvisory LinkedIn: [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*