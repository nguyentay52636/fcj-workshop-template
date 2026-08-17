---
title: "deploy.yml — Required Reviewer Issue and Fix"
date: 2026-08-09
weight: 2
chapter: false
pre: " <b> 5.9.2. </b> "
---

This is the most interesting page in the CI/CD section — not because the design is complex, but because the **first design did not behave as intended**, and finding + fixing that is a real story worth documenting.

#### Original design (did not work)

The `terraform-apply` job was meant to use a GitHub Environment `production` with **"Required reviewers"** — idea: merge to `main` would **auto-trigger**, then **pause for approval** in the Actions tab before a real apply.

```yaml
# Original design (did NOT behave as expected)
on:
  push:
    branches: [main]

jobs:
  terraform-apply:
    environment: production # expected: wait for approval here
    runs-on: ubuntu-latest
    steps:
      - run: terraform apply -auto-approve
```

#### Broke in practice

{{% notice warning %}}
📌 **Deploy ran immediately — no approval pause.** Checking with `gh repo view --json isPrivate` showed why: **"Required reviewers"** on a GitHub Environment is **only available for private repos in an Organization on a paid GitHub Team/Enterprise plan**. `RAGonAWS` is a **private personal-account** repo, so that option **does not appear in the UI at all** — not a misconfiguration, a product-tier limit.
{{% /notice %}}

This is the hardest class of bug to spot: no error message, the YAML is valid, only the **behavior** is wrong — you only catch it by watching Deploy run instantly and then digging with `gh repo view`.

#### Fix: switch to `workflow_dispatch`

```yaml
# .github/workflows/deploy.yml (current)
on:
  workflow_dispatch: {}

concurrency:
  group: deploy-production
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  terraform-apply:
    name: terraform apply (production)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: "~> 1.9"
          cli_config_credentials_token: ${{ secrets.TF_API_TOKEN }}
      - run: terraform init
      - run: terraform apply -auto-approve -input=false
```

Trigger changed from `push: branches: [main]` to `workflow_dispatch: {}` — **merging to `main` now does nothing for deploy**; you must open **Actions → Deploy → Run workflow** for a real apply.

The workflow also sets `concurrency: group: deploy-production` with `cancel-in-progress: false` so two applies never run at once against the same HCP Terraform workspace.

{{% notice note %}}
`workflow_dispatch` is the **free gate** that replaces Required reviewers, with the same intent: **no deliberate click → nothing gets applied**. You do not always need the “standard” feature — a simpler mechanism that meets the safety goal (no surprise auto-apply) is enough when the standard feature is unavailable.

**Before → after:** push to `main` used to auto-run apply (Required reviewer never appeared). After the fix, deploy only starts from the manual **Run workflow** button in the Actions tab.
{{% /notice %}}

#### Evidence on GitHub Actions

This is the correct **Deploy** screen with the **Run workflow** button (`workflow_dispatch`). The **small miss:** both runs are red — not a wrong trigger, but **`terraform init` failed** (usually missing/wrong `TF_API_TOKEN`); `apply` was skipped (~10s).

| Run | Trigger | Result |
| --- | --- | --- |
| #1 (3 Aug) | `push` — old design | failure at `terraform init` |
| #2 (today) | **Manually run** — current gate | failure at `terraform init` (same cause) |

<div align="center">

![GitHub Actions Deploy — Run workflow, both runs fail at init](/images/5-Workshop/5.9-CICD/deploy-run-workflow.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Figure 5.9.2. Deploy — Run workflow clicked (#2). Failed fast at <code>terraform init</code>; nothing applied on AWS. The manual gate is correct; check the HCP token in [5.9.3](../5.9.3-Manual-Setup-and-Scope-Limitations/).
</p>

</div>

---

#### Next Content

- [5.9.3 - Manual Setup and Scope Limitations](../5.9.3-Manual-Setup-and-Scope-Limitations/)
