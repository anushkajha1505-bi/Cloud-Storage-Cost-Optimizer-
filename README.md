# Cloud-Storage-Cost-Optimizer-
Built a Python-based analysis tool using Boto3 to audit AWS S3 bucket usage, identify redundant or oversized files, and generate cost reduction reports.
# Cloud Storage Cost Optimizer

A Python CLI tool that audits your AWS S3 buckets, analyzes object-level storage patterns, and generates a data-driven cost reduction report — all without modifying any of your actual data.

> **Portfolio project** demonstrating the intersection of cloud tooling (AWS Boto3) and data analysis (Pandas) — analyzing storage usage patterns the same way a FinOps analyst would approach infrastructure cost review.

---

##  The Problem

As cloud usage grows, S3 costs often balloon silently. Teams end up with:
- Buckets full of files that haven't been accessed in months
- Large objects that could be moved to cheaper storage tiers (Glacier, IA)
- No visibility into which buckets or file types are driving cost

This tool gives you a clear breakdown and actionable recommendations in minutes.

---

##  Project Structure

```
cloud-cost-optimizer/
│
├── src/
│   ├── audit.py              # Core S3 auditor using Boto3
│   ├── analyzer.py           # Pandas-based cost analysis logic
│   ├── report_generator.py   # Generates CSV + console summary
│   └── config.py             # Configuration (region, thresholds, pricing)
│
├── outputs/
│   ├── audit_report.csv      # Full object-level audit (generated)
│   └── cost_summary.txt      # Human-readable recommendations (generated)
│
├── tests/
│   └── test_analyzer.py      # Unit tests for the analysis logic
│
├── sample_data/
│   └── mock_s3_audit.csv     # Synthetic S3 audit data for offline testing
│
├── requirements.txt
└── README.md
```

---

##  Setup

### Prerequisites
- Python 3.9+
- AWS CLI configured (`aws configure`) OR AWS credentials in environment variables
- IAM permissions needed: `s3:ListAllMyBuckets`, `s3:ListObjectsV2`, `s3:GetBucketLocation`

### Installation

```bash
# 1. Clone the repo
git clone https://github.com/Anu-1505/cloud-cost-optimizer.git
cd cloud-cost-optimizer

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure AWS credentials (if not already)
aws configure
# OR export AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION
```

---

##  Usage

### Full Audit (Live AWS Account)

```bash
python src/audit.py
```

This scans all accessible S3 buckets and saves raw data to `outputs/audit_report.csv`.

### Analysis + Report (on audit output or sample data)

```bash
# Analyze live audit results
python src/analyzer.py --input outputs/audit_report.csv

# Test with sample data (no AWS needed!)
python src/analyzer.py --input sample_data/mock_s3_audit.csv
```

### Generate Full Report

```bash
python src/report_generator.py --input sample_data/mock_s3_audit.csv
```

---

## 📋 Requirements

```
boto3==1.34.14
pandas==2.1.4
numpy==1.26.3
tabulate==0.9.0
```

---

##  What the Analyzer Does

### 1. Bucket-Level Size Breakdown
Groups all objects by bucket and computes total storage size (GB), object count, and estimated monthly cost using S3 Standard pricing ($0.023/GB).

### 2. Cold Object Detection
Flags objects not modified in 90+ days as candidates for S3 Infrequent Access (IA) tier, which costs ~46% less than Standard.

### 3. Large Object Report
Lists the top 20 largest files with their bucket, size, and last modified date — these alone often account for 60–80% of storage cost.

### 4. File Type Analysis
Groups objects by extension to find which file types (`.log`, `.bak`, `.tmp`) are bloating storage.

### 5. Cost Recommendations
Generates a text summary estimating how much could be saved by:
- Moving cold objects to IA tier
- Archiving objects untouched for 180+ days to Glacier
- Deleting objects in clearly temp/cache buckets

---

##  Sample Report Output

```
=== AWS S3 COST OPTIMIZATION REPORT ===
Generated: 2026-06-28

Total Storage Analyzed:    847.3 GB across 12 buckets
Estimated Current Cost:    $19.49/month (S3 Standard)

--- RECOMMENDATIONS ---
 Move 312 GB of cold objects (90+ days) to S3-IA
   Estimated saving: ~$4.49/month (23%)

 Archive 89 GB untouched 180+ days to Glacier
   Estimated saving: ~$1.83/month (9.4%)

 Review 23 .tmp and .log files totalling 4.1 GB
   Likely safe to delete: ~$0.09/month savings

Total Potential Monthly Saving: ~$6.41 (32.9% reduction)
```

---

##  Running Tests (No AWS Needed)

```bash
python -m pytest tests/ -v
```

Tests use the synthetic `sample_data/mock_s3_audit.csv` so no AWS connection is needed.

---

##  Safety Notes

- This tool is **read-only**. It calls `ListObjectsV2` and `ListAllMyBuckets` only.
- It **never deletes, modifies, or moves** any objects.
- All output is local CSV/text files — nothing is written back to AWS.

---

## 📬 Contact

**Anushka Jha** — [anushkajha1505@gmail.com](mailto:anushkajha1505@gmail.com) | [LinkedIn](https://linkedin.com/in/anushka-jha-810319313)
