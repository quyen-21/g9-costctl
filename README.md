# g9-costctl

A high-performance, production-grade AWS resource and cost optimization CLI tool developed by Group 9 (G9) for resource tag enforcement, idle detection, EBS volume migration, and cost governance.

All core features and stretch goals are fully implemented with **25/25 automated tests passing**.

---

## Features & Implemented Commands

The `costctl` CLI provides powerful command-line utilities across the following areas:

### Core Capabilities
* **`list`**: Query and filter EC2 instances, RDS databases, S3 buckets, and EBS volumes by specific tags or missing tags.
* **`cost`**: Query the AWS Cost Explorer API to aggregate and report historical cost metrics grouped by service for resources matching a specified tag.
* **`tag`**: Seamlessly add or update tags on EC2, RDS, S3, or EBS volume resources.
* **`terminate`**: Safely stop or delete a resource with confirmation checks.

### Advanced Optimization & Governance (Stretch)
* **`clean`**: Bulk terminate resources matching a specific tag (runs in dry-run mode by default for maximum safety).
* **`idle`**: Scan running EC2 instances and analyze CloudWatch `CPUUtilization` metrics to discover idle resources.
* **`migrate-gp3`**: Analyze EBS volumes and migrate cheaper/faster gp2 volumes to gp3 (with live migration, zero downtime, dry-run & apply support).

---

## Installation & Setup

### Prerequisites
* Python 3.11+
* AWS credentials configured locally (`aws configure` or standard environment variables)

### Installation
```bash
# Clone the repository
git clone https://github.com/quyen-21/g9-costctl.git
cd g9-costctl

# Setup virtual environment and install dependencies
python -m venv .venv
.venv/Scripts/activate
pip install -r requirements.txt

# For development and testing dependencies
pip install -r requirements-dev.txt
```

---

## Command Usage & Examples

Every command supports standard options including `--region` to override the default AWS region.

```bash
# List EC2 instances with specific environment tags
./costctl.py list ec2 --tag Environment=dev

# List volumes missing an Application tag
./costctl.py list volume --missing-tag Application

# Calculate cost of Merxly application resources over the last 7 days
./costctl.py cost --tag Application=Merxly --days 7

# Tag an EC2 instance with Owner and Application metadata
./costctl.py tag ec2 --id i-0abc123456789def0 --set Owner=G9 --set Application=Merxly

# Identify idle EC2 instances with CPU utilization below 5% over 24 hours
./costctl.py idle --threshold 5.0 --hours 24

# Perform a dry-run migration check of gp2 volumes to gp3
./costctl.py migrate-gp3

# Apply the live, zero-downtime gp3 migration for all gp2 volumes
./costctl.py migrate-gp3 --apply
```

---

## Testing

The test suite leverages `pytest` and `moto` to simulate AWS services locally, ensuring full reliability without calling live AWS APIs.

To execute the test suite:
```bash
pytest -v tests/
```

Result: **25/25 passing tests**

---

## Operational Reflections & Review

Refer to [REFLECTIONS.md](file:///e:/code/REFLECTIONS.md) for an in-depth operational review and engineering lessons learned, covering:
1. **Cleanup blast-radius control** — safety measures to limit accidental damage
2. **Idle-resource detection vs Trusted Advisor** — when to use each approach
3. **W7 carry-over** — which commands should continue into multi-account operations
4. **Implementation review** — validation notes and improvement decisions

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
