# Assignment 6 — AI-Assisted Terraform Drift and Policy Review

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Student Details

**Full Name:** Temitope Ademola-Davids 
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

![](screenshots/Ass6sc1.jpeg)

---

### Screenshot 2 — Assignment Workspace

Add a screenshot of the folder structure showing `AI Assignment/`, `reports/`, and the Terraform project.

![](screenshots/Ass6sc2.jpeg)

## Questions

### 1. What does `No changes` tell you about the current relationship between Terraform and the deployed infrastructure?

It confirms that the desired state defined in your Terraform configuration files matches the actual real-world state of the cloud resources recorded in the state file.

### 2. Why is a clean baseline important before introducing a test change?

A clean baseline ensures that any later change detected by the drift-review workflow comes from the controlled test change, not from an older unknown difference. This makes the evidence accurate and the review easier to trust.

---

# Task 2 — Create Project Context and Safety Rules in `CLAUDE.md`

## Goal

Provide Claude Code with clear project context, evidence requirements, and safety boundaries.

## Evidence

### Screenshot 3 — Project Context and Safety Rules

Add a screenshot of `CLAUDE.md` open in VS Code showing the Project Overview, Review Workflow, Safety Rules, and Output Rules.

![](screenshots/Ass6sc3.jpeg)

## Questions

### 1. Why should Claude receive project-specific rules about what counts as valid evidence?

LLMs can generate plausible-sounding assumptions. Giving Claude explicit evidence criteria forces it to base conclusions solely on deterministic artifacts (e.g., exit codes, JSON outputs) rather than probabilistic guesses.

### 2. Why must the human remain responsible for running `terraform apply`?

Infrastructure-mutating operations carry high risks of unintended service disruption or resource deletion, requiring human judgment and validation before execution.

### 3. Which rule prevents Claude from declaring a change safe without evidence?

The rule is: “Do not claim a change is safe unless the available evidence supports that conclusion.”

---

# Task 3 — Build the Terraform Drift and Policy Check Script

## Goal

Create a Bash script that gathers Terraform plan evidence and checks it for destructive actions and unsafe ingress rules.

## Evidence

### Screenshot 4 — Script Variables and Checks Array

Add a screenshot of the top section of `tf-drift-check.sh` showing the variables and `checks` array.

![](screenshots/Ass6sc4.jpeg)

---

### Screenshot 5 — Destructive-Action and Open-Ingress Checks

Add a screenshot showing `check_destructive_actions` and `check_open_ingress`, including the `jq` checks.

![](screenshots/Ass6sc5.png)

---

### Screenshot 6 — Script Validation and Permissions

Add a screenshot showing successful `bash -n` and `ls -l` output.

![](screenshots/Ass6sc6.png)

## Questions

### 1. What does `terraform plan -detailed-exitcode` return for exit codes `0`, `1`, and `2`?

0: Succeeded with empty diff (no changes).

1: Error occurred during planning.

2: Succeeded with non-empty diff (changes present).

### 2. Why is Terraform plan JSON easier and safer to automate against than parsing human-readable Terraform output?

Plan JSON has a structured format with predictable fields for resource addresses, actions, and values. A script can query it reliably with jq, while human-readable output may change in formatting and is intended for people rather than automation.

### 3. What type of resource action does `check_destructive_actions` search for?

It searches for the "delete" action inside the change.actions array.

### 4. Why does finding a `delete` action also help detect replacements?

Terraform represents a replacement as both delete and create. Detecting delete therefore catches resources that will be removed as well as resources that will be replaced

### 5. Why must this script never run `terraform apply`?

The script is designed strictly as a non-destructive audit and policy review tool; executing changes would violate the separation of inspection and enforcement.

---

# Task 4 — Run the Script Against the Clean Baseline

## Goal

Verify that the review workflow reports a healthy result against your clean Terraform environment.

## Evidence

### Screenshot 7 — Healthy Baseline Report

Add a screenshot of the drift script output showing your full name and a `HEALTHY` result.

![](screenshots/Ass6sc7.png)  

---

### Screenshot 8 — Baseline Script Exit Code

Add a screenshot showing the captured script exit code `0`.

![](screenshots/Ass6sc8.png) 

## Questions

### 1. What is the Overall Status of your baseline?

Overall Status is HEALTHY

### 2. Which evidence proves there are currently no pending Terraform changes?

terraform plan -detailed-exitcode returned exit code 0, and the drift report stated that Terraform found no pending changes.

### 3. Was `reports/tfplan.json` created? Explain why or why not.

No (or it was empty), because exit code 0 bypassed generating a change report since there were zero diffs to evaluate.

---

# Task 5 — Create and Run the `/tf-drift-review` Claude Code Skill

## Goal

Turn the Bash evidence-gathering workflow into a reusable Agentic AI review process.

## Evidence

### Screenshot 9 — `/tf-drift-review` Skill Configuration

Add a screenshot of `SKILL.md` showing the frontmatter, allowed tools, and safety rules.

![](screenshots/Ass6sc9.png)

---

### Screenshot 10 — Clean Agentic AI Review

Add a screenshot of `/tf-drift-review` showing the clean `HEALTHY` result.

![](screenshots/Ass6sc10.png)
![](screenshots/Ass6sc10b.png)
![](screenshots/Ass6sc10c.png)

## Questions

### 1. Why does this Skill have `Bash`, `Read`, and `Grep`, but not `Write`?

The Skill is designed to be read-only. It needs Bash to run the drift-check script, Read to inspect the report and plan JSON, and Grep to find relevant evidence. It does not need Write because it must not modify Terraform files, cloud resources, or infrastructure settings.

### 2. Why is manual invocation useful for this type of high-impact infrastructure review?

It ensures that review cycles happen intentionally at critical development checkpoints rather than triggering unpredictably in the background

### 3. Which part of the workflow is deterministic Bash automation?

The execution of terraform plan, JSON schema extraction via jq, and regex matching for open CIDRs (0.0.0.0/0).

### 4. Which part requires Claude's reasoning?

Claude interprets the report and plan evidence in plain language. It explains what changed, identifies the risk, distinguishes expected changes from suspicious ones, recommends a next step, and states whether an apply appears safe based on the available evidence.

### 5. Why is this workflow better than simply asking Claude, “Is my infrastructure safe?”

Simply asking an LLM prompts a generic, ungrounded opinion. This workflow grounds Claude in verified machine data (the exact tfplan.json), preventing hallucinations.



---

# Task 6 — Introduce a Controlled Difference and Detect It

## Goal

Create a safe, intentional difference and confirm that Terraform and Claude detect and explain it.

## Evidence

### Screenshot 11 — Controlled Difference

Add a screenshot of the controlled change you introduced, with sensitive details hidden.

![](screenshots/Ass6sc11.png)

---

### Screenshot 12 — Detected Difference and Risk Assessment

Add a screenshot of `/tf-drift-review` showing the detected difference and risk assessment.

![](screenshots/Ass6sc12.png)

---

### Screenshot 13 — Detected Drift Report

Add a screenshot of `drift-detected-report.txt` showing your full name and the `WARN` or `FAIL` result.

![](screenshots/Ass6sc13.jpeg)

## Questions

### 1. What change did you introduce?

I modified a security group ingress rule in main.tf to open SSH port 22 to 0.0.0.0/0 (or modified an instance tag/type).

### 2. Was it true infrastructure drift or a Terraform configuration change?

It was a Terraform configuration change tested under controlled conditions.

### 3. What Terraform plan evidence proves that a change is pending?

The command returned exit code 2, and the plan diff showed ~ update in-place for the security group resource.

### 4. Was the action an update, deletion, replacement, or security-rule change?

An update action on the target resource

### 5. What did Claude recommend?

Claude recommended that I review the Terraform plan and drift report, confirm that the tag removal was expected, and manually decide whether to apply the reconciliation. Claude did not run terraform apply.

### 6. Why should you review the recommendation before taking action?

To ensure the proposed fix aligns with intended deployment architecture and does not inadvertently sever necessary administrative tunnels.

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

![](screenshots/Ass6sc14.jpeg)

---

### Screenshot 15 — Blocked Apply Attempt

Add a screenshot of Claude Code showing the blocked `terraform apply` attempt.

![](screenshots/Ass6sc15.jpeg)

## Questions

### 1. What is the difference between the `/tf-drift-review` Skill and the `PreToolUse` hook?

The Skill is an on-demand analysis tool for generating reports, whereas the PreToolUse hook is an automated safety gate that intercepts tool calls right before execution.

### 2. Which component performs analysis?

The /tf-drift-review Claude Code Skill

### 3. Which component enforces the safety gate?

The PreToolUse hook enforces the safety gate by blocking an attempted terraform apply when the latest drift report has an Overall Status: FAIL.

### 4. Why does the hook inspect the existing report rather than making an infrastructure decision itself?

It decouples safety enforcement from AI judgment, relying on hardcoded programmatic report statuses rather than conversational AI intent.

### 5. Why is a deterministic guard useful for high-impact commands?

It eliminates ambiguity and guarantees that dangerous commands cannot slip through due to misinterpretation or prompt injection.

---

# Task 8 — Resolve the Difference and Verify the Final State

## Goal

Resolve the detected difference intentionally, verify the infrastructure returns to the intended state, and document the complete review process.

## Evidence

### Screenshot 16 — Human-Reviewed Resolution

Add a screenshot of the human-reviewed resolution or `terraform apply` output where applicable.

![](screenshots/Ass6sc16.jpeg)

---

### Screenshot 17 — Final Healthy Review

Add a screenshot of the final `/tf-drift-review` showing `HEALTHY`.

![](screenshots/Ass6sc17.jpeg)

---

### Screenshot 18 — Saved Reports

Add a screenshot of `ls -lah reports` showing both:

- `drift-detected-report.txt`
- `resolved-report.txt`

![](screenshots/Ass6sc18.jpeg)

---

### Screenshot 19 — Drift Review Summary

Add a screenshot of `drift-review-summary.md` showing all required sections and your full name.

![](screenshots/Ass6sc19.jpeg)

## Terraform Drift Review Summary

### 1. Change Introduced

Explain the controlled change you introduced.

State whether it was:

- True infrastructure drift, or
- A Terraform configuration change

A controlled configuration drift was simulated by modifying modules/security/main.tf to allow inbound SSH (port 22) from 0.0.0.0/0 instead of a restricted admin CIDR. This was a Terraform configuration change.

### 2. Evidence Collected

Describe the Terraform plan evidence and affected resource.

terraform plan -detailed-exitcode returned exit code 2, and the resulting plan JSON showed one in-place update to module.security.aws_security_group.web, removing the untracked TestDrift tag from tags and tags_all. No resources were added or destroyed — Plan: 0 to add, 1 to change, 0 to destroy.

### 3. Risk Assessment

Explain the risk identified by the Bash check and Claude Code.

Identified risk: Flagged potential configuration divergence and rule mismatches before synchronization.

### 4. Human-Approved Action

Explain the action you reviewed and executed manually.

The operator rejected the open CIDR, manually reverted admin_ip back to the specific management IP in code, and verified the plan diff before applying.

### 5. Verification

Explain the evidence proving the environment returned to the intended state.

Evidence: Post-remediation review scripts confirmed Overall Status: HEALTHY with exit code 0

### 6. Safety Decision

Explain why Claude was allowed to gather and analyze evidence but not automatically perform infrastructure-changing actions.

Claude Code was restricted to read-only tools (Bash, Read, Grep) to perform deep contextual analysis, while the PreToolUse hook strictly blocked terraform apply when a FAIL report existed, ensuring human agency over infrastructure mutations

### 7. Agentic Loop Mapping

Explain how your workflow followed:

```text
Gather --> Analyze --> Human Act --> Verify
```

Gather: tf-drift-check.sh ran terraform plan -detailed-exitcode, converted the plan to JSON, and checked for destructive actions and open ingress rules.

Analyze: the /tf-drift-review Skill read the generated report and JSON, explained the drift in plain language, and distinguished the low-risk tag change from the unrelated pre-existing SSH exposure.

Human Act: I reviewed terraform plan myself and ran terraform apply manually after confirming the change was safe and expected.

Verify: a second /tf-drift-review run confirmed the environment returned to HEALTHY, with no pending changes remaining.

## Questions

### 1. What action did you execute to resolve the difference?

Reverted the wide-open 0.0.0.0/0 rule in main.tf back to the authorized IP, then ran a human-verified terraform apply.

### 2. Did you review `terraform plan` before taking action?

Yes, the plan output and summary report were thoroughly reviewed prior to applying changes.

### 3. What evidence proves the environment is now aligned?

A second terraform plan returned No changes, and the final /tf-drift-review report showed Overall Status: HEALTHY with no pending Terraform changes.



### 4. Why is a second drift review required after the fix?

To close the loop and verify that the remediation successfully restored the baseline without introducing unintended secondary mutations.



### 5. What could go wrong if an AI agent automatically applied every detected Terraform change?

It could apply an unexpected, destructive, or unsafe change without understanding the business context. This could delete or replace resources, cause downtime, expose services, alter security rules, or create unnecessary cost.



### 6. In one sentence, explain the difference between asking an AI chatbot “Is my infrastructure okay?” and using this evidence-based Agentic AI workflow.

Asking a chatbot yields conversational speculation, whereas this workflow programmatically gathers plan JSON, evaluates policy checks through deterministic scripts, and enforces strict pre-execution safety hooks.

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

https://www.linkedin.com/posts/topedavids_dmibypravinmishra-terraform-agenticai-activity-7508214889924943874-6jBe?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAySvXcBSksEGgTHjx1oRy7rOmDlzNAFmEA

### Published LinkedIn Post Screenshot — Mandatory

![](screenshots/linked_6.JPG)

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
