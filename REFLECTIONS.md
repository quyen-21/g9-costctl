# REFLECTIONS.md

## 1. `clean --apply` blast radius

If we accidentally run `clean --tag Environment=dev --apply` in a shared AWS account, it could terminate resources owned by other teams who also tag their resources with `Environment=dev`. To limit damage, we should:

- **Require an additional owner tag** like `Owner=G9` to narrow the scope and avoid affecting other teams' resources.
- **Keep dry-run as default** (already implemented) — this forces the operator to review what will be deleted before committing.
- **Add a confirmation prompt** even with `--apply`, showing the count of resources that will be terminated and requiring explicit `y` input.
- **Restrict IAM permissions** using tag-based conditions (e.g., `ec2:ResourceTag/Owner=G9`) so the CLI physically cannot terminate resources it doesn't own.
- **Log every destructive action** to CloudTrail or a local audit file for post-incident review.

## 2. `idle` vs Trusted Advisor

The `idle` command is useful for **quick, on-demand checks** because it can scan CPU over a configurable window (e.g., 1–24 hours) and show possible waste immediately. This is great during a short W6 demo or when you need fast feedback after launching test instances.

**Trusted Advisor** is more reliable for **longer-term recommendations** because it uses a 14-day window with more metrics (network, disk I/O) beyond just CPU. However, it may not be fast enough during a short demo window, and it only updates every few hours.

**When to trust `idle` more**: Short-lived test environments, quick spot checks, or when you need results within minutes.

**When to trust Trusted Advisor more**: Production workloads, steady-state environments, or when you need a holistic view including network and disk patterns — not just CPU.

## 3. W7 carry-over

We will keep the following commands for W7 (production-style multi-account):

- **`list`** — Essential for identifying ownership and untagged resources across accounts.
- **`cost`** — Critical for cost attribution and chargeback reporting per team/application.
- **`tag`** — Needed to enforce tagging governance and fix compliance gaps.
- **`idle`** — Useful for continuous waste detection across accounts.

We would **modify** `terminate` and `clean` to add stricter safeguards:
- Require MFA or approval workflow before `--apply`.
- Add account-level allow-lists to prevent accidental cross-account damage.
- Add `--account` flag for multi-account targeting with assume-role.

We would **drop** `migrate-gp3` as a CLI command and instead implement it as an **automated policy** (e.g., AWS Config rule + Lambda) that detects and migrates gp2 volumes without manual intervention.

## 4. Implementation review

During implementation, we focused on validating boto3 API usage, matching the expected command behavior, and checking safety boundaries for commands that can modify AWS resources.

The most important review points were:

- **S3 tagging merge logic** — ensuring `put_bucket_tagging` merges new tags with existing tags instead of accidentally replacing the full tag set.
- **Error handling boundaries** — deciding where to catch `ClientError` so command failures show friendly messages instead of raw stack traces.
- **Output formatting** — matching exact strings expected by the tests, such as `"Refusing"`, `"Terminated"`, `"AWS error"`, and `"dry-run"`.
- **State filtering in `clean`** — ensuring terminated or shutting-down instances are skipped, and only `available` EBS volumes are selected for deletion.
- **Destructive action safeguards** — keeping `clean` as dry-run by default and requiring explicit flags before making changes.

These review steps helped make the CLI safer for workshop use while keeping the implementation simple enough to explain during W6 Q&A.
