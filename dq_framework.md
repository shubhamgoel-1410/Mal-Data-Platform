## Overall Flow

                Silver Tables
                     │
                     ▼
             DQ Rule Engine
                     │
      ┌──────────────┴──────────────┐
      │                             │
      ▼                             ▼
Passed Records               Failed Records
      │                             │
      ▼                             ▼
Trusted Silver            Quarantine Table
      │
      ▼
Generate DQ Metrics
      │
      ▼
dq_check_result
      │
      ▼
Alerting

## 1. DQ Rule Configuration
### `dq_rule_config - Table Schema`

| column_name   | data_type | description |
| ------------- | --------- | ----------- |
| rule_id       | INTEGER   | Primary key |
| table_name    | STRING    | Table name  |
| column_name   | STRING    | Column name |
| rule          | STRING    | Rule        |
| severity      | STRING    | Severity    |
| threshold     | FLOAT     | Threshold   |

---

## 2. Execute Rules

for rule in dq_rule_config:

    failed_records = execute(rule)

    if rule.severity == "CRITICAL":
        move_to_quarantine(failed_records)

    else:
        log_warning(failed_records)


---

## 3. DQ Result Table - Table schema
### `dq_check_result`

| column_name   | data_type | description |
| ------------- | --------- | ----------- |
| run_date      | DATE      | Run date    |
| dataset       | STRING    | Dataset name|
| total_records | INTEGER   | Total records|
| passed        | INTEGER   | Passed records|
| failed        | INTEGER   | Failed records|
| critical_failures | INTEGER   | Critical failures|
| warnings      | INTEGER   | Warnings|


## 4. Alerting

if pass_rate < 99:

    send_slack()

if critical_failures > 0:

    send_email()

if duplicate_rate > 2:
    create_incident()


Some sample DQ checks:
Customer Profile
1. UUID is not null (mandatory identifier)
2. Emirates ID is unique and valid
3.No duplicate customer records (UUID)

AECB (Credit Bureau)
1. Emirates ID is not null
2. Credit score is between 300–900
3. Report date is within the last 30 days (freshness)
4. No duplicate report for the same customer and report date

Fraud Detection
1. Phone and Email are not null
2. Fraud score is within the valid range
3. No duplicate fraud events (Event ID)

AML / PEP
1. Customer Name and DOB are not null
2. AML Status is one of (CLEAR, MATCH, REVIEW)
3. No duplicate webhook events
4. Screening result is within SLA (e.g., last 24 hours)

Cross-Source Checks (Most Important)
1. Every record should resolve to a single customer_id
2. One customer should have only one Emirates ID
3. No duplicate customers after identity resolution

