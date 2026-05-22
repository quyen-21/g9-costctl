# g9-costctl — XBrain W6 side challenge

A small AWS-resource-management CLI built by Group 9 (G9) for the XBrain W6 side challenge. **All command logic has been implemented and all 25/25 tests pass.**

> **Side challenge is OPTIONAL and does NOT count toward W6 score or bonus cap.**
> Recognition is separate (Slack callout / Phase 2 selection / portfolio).
> See `outputs/W6/W6_downtime_exercises.md` in the XBrain program repo for
> the full brief.

---

## Test Result

**Final test result: 25/25 passing.**

Implemented commands:
- `list` — List EC2/RDS/S3/Volume with tag filtering (7 tests)
- `terminate` — Terminate/delete resources with confirmation (4 tests)
- `tag` — Add/update tags on resources (manual verify)
- `cost` — Cost Explorer query by tag (manual verify)
- `clean` — Bulk terminate by tag with dry-run (4 tests)
- `idle` — Find idle EC2 by CPU average (manual verify)
- `migrate-gp3` — gp2 → gp3 EBS migration planner (manual verify)

---

## What's given vs what you build

| Provided (don't reinvent) | Your job |
|---------------------------|----------|
| `costctl.py` — argparse entrypoint, dispatch table | Implement each command's `run(args)` |
| `commands/_common.py` — `parse_kv`, `tags_to_dict`, `tags_match`, `confirm` | Use these helpers, don't rebuild them |
| `tests/test_common.py` — 10 unit tests for the helpers (all green) | Don't modify — they verify the helpers still work |
| `tests/test_list.py`, `tests/test_terminate.py`, `tests/test_clean.py` — failing tests that define each command's behavior | Make them green |
| Module docstrings in every `commands/*_cmd.py` — full spec, hints, AWS APIs to use | Read them carefully before coding |
| `Makefile`, `requirements*.txt`, `.gitignore`, `LICENSE` | Untouched |
| `sample_output/*.txt` | Real outputs from running against an AWS account |

**Initial state of the starter repo:** 10 passed (helpers), 15 failed (commands).
**Final state of this G9 repo:** 25/25 passing.

---

## Quickstart

```bash
# 1. Clone the completed G9 repo
git clone https://github.com/quyen-21/g9-costctl.git && cd g9-costctl

# 2. Install dependencies
make install-dev                   # or: pip install -r requirements-dev.txt

# 3. Run tests — final expected result: 25/25 passing
make test

# 4. Confirm the CLI help works
./costctl.py --help
```

Configure AWS credentials when you're ready to run against your account:

```bash
aws configure                      # or set AWS_* env vars
./costctl.py list ec2              # lists EC2 instances
```

---

## Implementation roadmap

Recommended order. You need `list` + at least 2 of the next 3.

### Required

| # | File | Make pass | Time |
|---|------|-----------|------|
| 1 | `commands/list_cmd.py` | `pytest tests/test_list.py` (7 tests) | ~45 min |
| 2 | Pick **at least 2** of: | | |
|   | • `commands/cost_cmd.py` | Manual verify with `./costctl.py cost --tag X=Y --days 7` and compare to AWS Console | ~30 min |
|   | • `commands/terminate_cmd.py` | `pytest tests/test_terminate.py` (4 tests) | ~40 min |
|   | • `commands/tag_cmd.py` | Manual verify with `tag` + `list` roundtrip | ~30 min |

### Stretch (optional — extra portfolio value)

| File | Make pass | Time |
|------|-----------|------|
| `commands/clean_cmd.py` | `pytest tests/test_clean.py` (4 tests) | ~30 min |
| `commands/idle_cmd.py` | Manual verify | ~45 min |
| `commands/migrate_gp3_cmd.py` | Manual verify, then run `--apply` once for real if safe | ~30 min |

### How to read a command file

Every `commands/*_cmd.py` starts with a module docstring that includes:

- **WHAT YOU MUST BUILD** — high-level behavior
- **HELPERS YOU CAN USE** — point to `commands/_common.py`
- **AWS APIS YOU'LL NEED** — exact boto3 calls
- **EXPECTED OUTPUT FORMAT** — copy this exactly when you `print(...)`
- **VERIFY** — pytest command or manual recipe

---

## Commands

| Command | What it does | Tier |
|---------|--------------|------|
| `list <type>` | List EC2/RDS/S3/Volume, filter by tag or missing-tag | core |
| `cost --tag k=v` | Sum cost over N days for resources matching a tag | core |
| `terminate <type> --id` | Terminate/delete one resource (asks confirmation) | core |
| `tag <type> --id --set` | Add/update tags on one resource | core |
| `clean --tag k=v` | Bulk terminate resources by tag (dry-run by default) | stretch |
| `idle` | Find idle EC2 by CPU average | stretch |
| `migrate-gp3` | Plan or apply gp2 → gp3 EBS migration | stretch |

Resource types: `ec2`, `rds`, `s3`, `volume`.

### Example invocations

```bash
# List
./costctl.py list ec2 --tag Environment=dev
./costctl.py list ec2 --missing-tag Application
./costctl.py list s3

# Cost
./costctl.py cost --tag Application=Merxly --days 7

# Terminate
./costctl.py terminate ec2 --id i-0abc123
./costctl.py terminate ec2 --id i-0abc123 --force

# Tag
./costctl.py tag ec2 --id i-0abc --set Owner=alice --set Application=Merxly

# Stretch
./costctl.py clean --tag purpose=practice          # dry-run
./costctl.py clean --tag purpose=practice --apply
./costctl.py idle --threshold 5 --hours 24
./costctl.py migrate-gp3
./costctl.py migrate-gp3 --apply --volume-id vol-0xyz
```

---

## Requirements

- Python 3.11+
- `boto3` via `make install`
- AWS credentials with:
  - **Read**: EC2, RDS, S3, CloudWatch, Cost Explorer
  - **Write** only for `terminate` / `tag` / `clean` / `migrate-gp3`: EC2, RDS, S3

For tests:
- `moto`, `pytest`, `pytest-cov` via `make install-dev`

---

## Project structure

```
g9-costctl/
├── costctl.py                # argparse entrypoint
├── commands/
│   ├── _common.py            # helpers — implemented
│   ├── list_cmd.py           # implemented
│   ├── cost_cmd.py           # implemented
│   ├── terminate_cmd.py      # implemented
│   ├── tag_cmd.py            # implemented
│   ├── clean_cmd.py          # implemented (stretch)
│   ├── idle_cmd.py           # implemented (stretch)
│   └── migrate_gp3_cmd.py    # implemented (stretch)
├── tests/                    # all provided tests pass
│   ├── conftest.py
│   ├── test_common.py        # 10 tests
│   ├── test_list.py          # 7 tests
│   ├── test_terminate.py     # 4 tests
│   └── test_clean.py         # 4 tests
├── sample_output/            # real command outputs
├── REFLECTIONS.md            # reflection answers
├── Makefile
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
└── README.md
```

---

## Verification commands

```bash
pytest tests/test_common.py -v
pytest tests/test_list.py -v
pytest tests/test_terminate.py -v
pytest tests/test_clean.py -v
make test
```

---

## Reflections

See `REFLECTIONS.md` for operational review and lessons learned, including:

1. **Cleanup blast-radius control** — safety measures to limit accidental damage
2. **Idle-resource detection vs Trusted Advisor** — when to use each approach
3. **W7 carry-over** — which commands should continue into multi-account operations
4. **Implementation review** — validation notes and improvement decisions

---

## Submission checklist (W6 side challenge)

- [x] Fork → rename to `g9-costctl` → clone locally
- [x] `make install-dev && make test` baseline understood: starter starts at 10 passed / 15 failed
- [x] Implement `list` → `pytest tests/test_list.py` all green
- [x] Implement ≥ 2 of (`cost`, `terminate`, `tag`) — all 3 implemented
- [x] Optional stretch: `clean`, `idle`, and `migrate-gp3` implemented
- [x] `make test` final score reported in README: **25/25 passing**
- [x] Generate real outputs from AWS account in `sample_output/*.txt`
- [x] `REFLECTIONS.md` with 4 answers
- [x] At least 3 meaningful commits (init → first command working → final polish)
- [x] Replace `g<N>` placeholders throughout README with G9
- [x] Add Team section with member names
- [x] Tag: `git tag w6-sidechallenge-v1 && git push --tags`
- [x] Prepare Slack reply format below

Submission message:

```text
G9 — https://github.com/quyen-21/g9-costctl — 25/25 tests passing — implemented: list, cost, terminate, tag, clean, idle, migrate-gp3
```

Reminder: **OPTIONAL and does NOT count toward W6 score.** Recognition is
separate (Slack callout / Phase 2 selection / portfolio).

---

## License

MIT — see `LICENSE`.

---

## Team

- Trần Văn Đức
- Nguyễn Hữu Định
- Trần Đình Bảo Long
- Nguyễn Đức Chinh
- Lê Duy Khánh
- Trương Thị Mỹ Quyên
- Hoàng Trọng Tấn
- Lê Hoàng Trung Kiên

---

*G9 costctl — XBrain W6 side challenge submission.*
