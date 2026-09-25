# Part 2 run sheet (15 minutes)

The brief's six requirements, in the order the panel will see them: customer problem and ideal customer, discovery, value pillars, workflow, outcomes, objection.

## Before the session

- [ ] `aws login --profile jim-sandbox`
- [ ] Run `scripts/stage-demo.sh`, so the failing PR already exists and Checkov has already failed on it.
- [ ] Browser tabs, in order:
  1. The slide deck
  2. `payments-infra` → `deploy.yml`
  3. The open PR
  4. `platform-workflows` → `terraform.yml`
  5. Repo → Settings → Secrets (empty)
  6. Settings → Actions → General (the allow-list)
  7. AWS IAM → role `gha-payments-infra-apply` → Trust relationships
  8. CloudTrail → Event history, filtered to `AssumeRoleWithWebIdentity`
- [ ] Backup: a screen recording of the full cycle.

## Timeline

| Time | Slide or screen | Beat |
| --- | --- | --- |
| 0:00 | Cover | The intro line from the speaker notes. Hand the panel their roles. |
| 0:30 | Who this is for | The customer and the problem, about a minute. |
| 1:30 | Discovery | Ask the four questions and note their answers. Say which pillar you'll focus on. |
| 3:30 | Pillars | Start with the pillar their answers pointed to. |
| 5:00 | Workflow map | 10 seconds, then switch to the browser. |
| 5:15 | `deploy.yml` | "This is the Payments team's whole pipeline. Everything else is owned by the platform team." |
| 6:00 | The PR | The policy check is red and the merge is blocked. Open the Checkov output and read one failed check aloud. |
| 7:00 | Fix | Edit `main.tf` on the PR branch in the GitHub editor: set both flags back to `true` and commit. "A policy for one named role isn't public, so the guardrail pointed to the correct fix." |
| 7:30 | While checks run | Show the Secrets page (empty) and the IAM trust policy (only this repo's `production` environment). **This covers the runner wait time.** |
| 9:00 | The PR | Green checks, and the plan comment shows 1 resource to add. Merge. |
| 9:30 | Actions run | The deploy is waiting for approval. Approve it, then watch the OIDC step and the apply. |
| 11:00 | CloudTrail | The `AssumeRoleWithWebIdentity` event with session name `gha-apply-<run id>`. "That's your security team's evidence, produced automatically." |
| 12:00 | Outcomes | Use their numbers from discovery, and only the rows they cared about. |
| 13:30 | Objection | Show the pinned SHA in `terraform.yml` and the Actions allow-list settings page. |
| 14:30 | Pilot | Finish with "Which team would you start with?" |

## If something breaks

- **Runner slow to start:** keep talking through the trust policy. It's rarely more than a minute.
- **Apply fails:** "This is exactly why the plan is on the PR." Then switch to the recording from step 4 onwards.
- **They interrupt with questions:** good. Answer, then say "let me show you where that lives" and go back to the workflow. Drop the outcomes table before you drop the objection.

## Honest framing, if asked "have you sold this?"

"I haven't sold GitHub Actions as a product. I've used it to run delivery on large AWS platforms and designed pipelines around it for client platforms, so this is how I'd position it to a customer like the ones I work with."
