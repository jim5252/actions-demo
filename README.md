# Part 2 demo: GitHub Actions secure delivery

Two public repos on a personal GitHub account, plus one AWS account.

| Folder | Becomes | Owned by (in the story) |
| --- | --- | --- |
| `platform-workflows/` | GitHub repo `platform-workflows` | Platform team: the standard pipeline |
| `payments-infra/` | GitHub repo `payments-infra` | Payments team: their Terraform plus a 6-line workflow |
| `bootstrap/` | Run once, locally | OIDC provider, state bucket, the two pipeline roles, and a stand-in vendor role |
| `scripts/stage-demo.sh` | Run before the demo | Opens the planted PR that should fail |

Both repos are public, because on GitHub Free, environment approvals only work on public repos. That's safe here: with OIDC there are no secrets to leak.

## One-time setup (about 45 minutes)

**1. Bootstrap AWS**

```bash
aws login --profile jim-sandbox && export AWS_PROFILE=jim-sandbox
cd bootstrap
terraform init
terraform apply -var github_owner=<your-github-username>
```

Keep the two role ARNs it outputs. If the state bucket name is already taken, change `state_bucket` here and in `payments-infra/versions.tf`.

**2. Create `platform-workflows`**
- Push the folder to a new public repo.
- Tag it: `git tag v1 && git push origin v1`.

**3. Create `payments-infra`**
- In `.github/workflows/deploy.yml`, replace `GITHUB_OWNER` and `AWS_ACCOUNT_ID`.
- Push to a new public repo.

**4. Settings in `payments-infra`**
- **Environments → New environment `production`:**
  - Required reviewers: yourself.
  - Deployment branches: `main` only.
- **Rules → Rulesets → New branch ruleset on `main`:**
  - Require a pull request.
  - Require status checks: `terraform / policy` and `terraform / plan`. These only appear in the list after one run has happened, so open a test PR first.
- **Actions → General → Actions permissions:**
  - Choose "Allow owner, and select non-owner, actions".
  - Allow actions created by GitHub.
  - Add `hashicorp/setup-terraform@*, aws-actions/configure-aws-credentials@*` to the allow-list.
  - If your account offers the option to require full-length commit SHA pinning, turn it on. This setting is your live answer to the objection.

**5. First deploy**
- Merge a trivial PR, then approve the `production` deployment.
- Check the bucket `payments-statements-<account-id>` now exists.

**6. Dress rehearsal**
- Run `scripts/stage-demo.sh` and confirm the policy check fails.
- Fix the change, merge and approve.
- Find `AssumeRoleWithWebIdentity` in CloudTrail event history (eu-west-2).
- Reset for the real demo.

## Checks I've already run

- Checkov 3.3.19 against `payments-infra`: the good config passes all 7 baseline checks.
- The planted PR adds a vendor bucket policy and relaxes the public access block. It fails 3 checks: `CKV_AWS_54`, `CKV_AWS_56` and `CKV2_AWS_6`.
- Setting the two flags back to `true` passes all 7 again, with the vendor policy kept. A policy granting access to one named role isn't public, so the block never needed relaxing. That's the point you make live.
- Actions are pinned to the commit SHAs of their latest releases as of 25 September 2026.
- Not run end to end: GitHub and AWS weren't reachable from my environment, so the dress rehearsal in step 6 is how you'll know it all works.

## Cost

Close to zero:
- Public repos have free Actions minutes.
- Empty S3 buckets cost nothing.
- KMS uses the AWS-managed key, charged per request (pennies).

## Tear down after the interview

```bash
cd payments-infra && terraform destroy
cd ../bootstrap && terraform destroy
```

`payments-infra` destroy needs AWS credentials locally, and the state bucket must be empty before bootstrap can delete it.
